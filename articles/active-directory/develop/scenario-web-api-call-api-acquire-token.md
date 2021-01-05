---
title: Obtenir un jeton pour une API web appelant des API web | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment générer une API web qui appelle des API web nécessitant l’acquisition d’un jeton pour l’application.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: e8301a1961479f57528802e6d8c0f10ceb0569d5
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558264"
---
# <a name="a-web-api-that-calls-web-apis-acquire-a-token-for-the-app"></a>Une API web qui appelle des API web : Acquérir un jeton pour l’application

Après avoir généré un objet d’application cliente, utilisez-le pour acquérir un jeton qui vous servira à appeler une API web.

## <a name="code-in-the-controller"></a>Code dans le contrôleur

### <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

*Microsoft.Identity.Web* ajoute des méthodes d’extension qui fournissent des services pratiques pour appeler Microsoft Graph ou une API web en aval. Ces méthodes sont expliquées en détail dans [Une API web qui appelle des API web : Appeler une API](scenario-web-api-call-api-call-api.md). Avec ces méthodes d’assistance, vous n’avez pas besoin d’acquérir manuellement un jeton.

Toutefois, si vous souhaitez acquérir manuellement un jeton, le code suivant montre un exemple d’utilisation de *Microsoft.Identity.Web* pour le faire dans un contrôleur d’API. Il appelle une API en aval nommée *todolist*.
Pour obtenir un jeton permettant d’appeler l’API en aval, injectez le service `ITokenAcquisition` par injection de dépendance dans le constructeur de votre contrôleur (ou votre constructeur de page, si vous utilisez Blazor), puis utilisez-le dans vos actions de contrôleur, en obtenant un jeton pour l’utilisateur (`GetAccessTokenForUserAsync`) ou pour l’application elle-même (`GetAccessTokenForAppAsync`) dans le cas d’un scénario basé sur un démon.

```csharp
[Authorize]
public class MyApiController : Controller
{
    /// <summary>
    /// The web API will accept only tokens 1) for users, 2) that have the `access_as_user` scope for
    /// this API.
    /// </summary>
    static readonly string[] scopeRequiredByApi = new string[] { "access_as_user" };

     static readonly string[] scopesToAccessDownstreamApi = new string[] { "api://MyTodolistService/access_as_user" };

    private readonly ITokenAcquisition _tokenAcquisition;

    public MyApiController(ITokenAcquisition tokenAcquisition)
    {
        _tokenAcquisition = tokenAcquisition;
    }

    public IActionResult Index()
    {
        HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);

        string accessToken = _tokenAcquisition.GetAccessTokenForUserAsync(scopesToAccessDownstreamApi);
        return await callTodoListService(accessToken);
    }
}
```

Pour plus d’informations sur la méthode `callTodoListService`, consultez [Une API web qui appelle des API web : Appeler une API](scenario-web-api-call-api-call-api.md).

### <a name="java"></a>[Java](#tab/java)

Voici un exemple de code appelé dans les actions des contrôleurs d’API. Il appelle l'API situé en aval, Microsoft Graph.

```java
@RestController
public class ApiController {

    @Autowired
    MsalAuthHelper msalAuthHelper;

    @RequestMapping("/graphMeApi")
    public String graphMeApi() throws MalformedURLException {

        String oboAccessToken = msalAuthHelper.getOboToken("https://graph.microsoft.com/.default");

        return callMicrosoftGraphMeEndpoint(oboAccessToken);
    }

}
```

### <a name="python"></a>[Python](#tab/python)
 
Une API web Python doit utiliser un middleware pour valider le jeton du porteur reçu du client. L’API web peut alors obtenir le jeton d’accès de l’API située en aval à l’aide de la bibliothèque MSAL Python en appelant la méthode [`acquire_token_on_behalf_of`](https://msal-python.readthedocs.io/en/latest/?badge=latest#msal.ConfidentialClientApplication.acquire_token_on_behalf_of).
 
Voici un exemple de code qui acquiert un jeton d’accès à l’aide de la méthode `acquire_token_on_behalf_of` et du framework Flask. Il appelle l’API en aval (le point de terminaison des abonnements de gestion Azure).
 
```python
def get(self):
 
        _scopes = ["https://management.azure.com/user_impersonation"]
        _azure_management_subscriptions_uri = "https://management.azure.com/subscriptions?api-version=2020-01-01"
 
        current_access_token = request.headers.get("Authorization", None)
        
        #This example only uses the default memory token cache and should not be used for production
        msal_client = msal.ConfidentialClientApplication(
                client_id=os.environ.get("CLIENT_ID"),
                authority=os.environ.get("AUTHORITY"),
                client_credential=os.environ.get("CLIENT_SECRET"))
 
        #acquire token on behalf of the user that called this API
        arm_resource_access_token = msal_client.acquire_token_on_behalf_of(
            user_assertion=current_access_token.split(' ')[1],
            scopes=_scopes
        )
 
        headers = {'Authorization': arm_resource_access_token['token_type'] + ' ' + arm_resource_access_token['access_token']}
 
        subscriptions_list = req.get(_azure_management_subscriptions_uri), headers=headers).json()
 
        return jsonify(subscriptions_list)
```

## <a name="advanced-accessing-the-signed-in-users-token-cache-from-background-apps-apis-and-services"></a>(Avancé) Accès au cache de jeton de l’utilisateur connecté à partir d’applications en arrière-plan, d’API et de services

[!INCLUDE [advanced-token-caching](../../../includes/advanced-token-cache.md)]

---

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Appeler une API](scenario-web-api-call-api-call-api.md).
