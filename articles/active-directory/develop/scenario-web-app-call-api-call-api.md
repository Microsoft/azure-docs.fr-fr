---
title: Appeler une API web à partir d'une application web | Azure
titleSuffix: Microsoft identity platform
description: Apprendre à générer une application web qui appelle des API web (appelant une API web protégée)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/25/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 016a2917c02deaecf35c526426cb2a6dbc89e338
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173187"
---
# <a name="a-web-app-that-calls-web-apis-call-a-web-api"></a>Application web appelant des API web : Appeler une API web

Maintenant que vous avez un jeton, vous pouvez appeler une API web protégée. Vous appelez généralement une API située en aval à partir du contrôleur ou des pages de votre application web.

## <a name="call-a-protected-web-api"></a>Appeler une API web protégée

L’appel d’une API web protégée dépend du langage et de l’infrastructure de votre choix :

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Lorsque vous utilisez *Microsoft.Identity.Web*, vous disposez de trois options pour appeler une API :

- [Option n°1 : Appeler Microsoft Graph à l'aide du kit de développement logiciel (SDK) Microsoft Graph](#option-1-call-microsoft-graph-with-the-sdk)
- [Option n°2 : Appeler une API web située en aval avec la classe d'assistance](#option-2-call-a-downstream-web-api-with-the-helper-class)
- [Option 3 : Appeler une API web située en aval sans la classe d'assistance](#option-3-call-a-downstream-web-api-without-the-helper-class)

#### <a name="option-1-call-microsoft-graph-with-the-sdk"></a>Option 1 : Appeler Microsoft Graph à l'aide du SDK

Vous souhaitez appeler Microsoft Graph. Dans ce scénario, vous avez ajouté `AddMicrosoftGraph` dans le fichier *Startup.cs* comme spécifié dans [Configuration de code](scenario-web-app-call-api-app-configuration.md#option-1-call-microsoft-graph), et vous pouvez injecter directement le `GraphServiceClient` dans votre contrôleur ou constructeur de page pour l'utiliser dans les actions. L'exemple de page Razor suivant affiche la photo de l'utilisateur connecté.

```CSharp
[Authorize]
[AuthorizeForScopes(Scopes = new[] { "user.read" })]
public class IndexModel : PageModel
{
 private readonly GraphServiceClient _graphServiceClient;

 public IndexModel(GraphServiceClient graphServiceClient)
 {
    _graphServiceClient = graphServiceClient;
 }

 public async Task OnGet()
 {
  var user = await _graphServiceClient.Me.Request().GetAsync();
  try
  {
   using (var photoStream = await _graphServiceClient.Me.Photo.Content.Request().GetAsync())
   {
    byte[] photoByte = ((MemoryStream)photoStream).ToArray();
    ViewData["photo"] = Convert.ToBase64String(photoByte);
   }
   ViewData["name"] = user.DisplayName;
  }
  catch (Exception)
  {
   ViewData["photo"] = null;
  }
 }
}
```

#### <a name="option-2-call-a-downstream-web-api-with-the-helper-class"></a>Option n°2 : Appeler une API web située en aval avec la classe d'assistance

Vous souhaitez appeler une API web autre que Microsoft Graph. Vous avez ici ajouté `AddDownstreamWebApi` dans le fichier *Startup.cs* comme spécifié dans [Configuration de code](scenario-web-app-call-api-app-configuration.md#option-2-call-a-downstream-web-api-other-than-microsoft-graph), et vous pouvez injecter directement un service `IDownstreamWebApi` dans votre contrôleur ou constructeur de page pour l'utiliser dans les actions :

```CSharp
[Authorize]
[AuthorizeForScopes(ScopeKeySection = "TodoList:Scopes")]
public class TodoListController : Controller
{
  private IDownstreamWebApi _downstreamWebApi;
  private const string ServiceName = "TodoList";

  public TodoListController(IDownstreamWebApi downstreamWebApi)
  {
    _downstreamWebApi = downstreamWebApi;
  }

  public async Task<ActionResult> Details(int id)
  {
    var value = await _downstreamWebApi.CallWebApiForUserAsync(
      ServiceName,
      options =>
      {
        options.RelativePath = $"me";
      });
      return View(value);
  }
}
```

`CallWebApiForUserAsync` possède également des remplacements génériques fortement typés qui vous permettent de recevoir directement un objet. Par exemple, la méthode suivante reçoit une instance `Todo`, qui est une représentation fortement typée du code JSON renvoyé par l'API web.

```CSharp
    // GET: TodoList/Details/5
    public async Task<ActionResult> Details(int id)
    {
        var value = await _downstreamWebApi.CallWebApiForUserAsync<object, Todo>(
            ServiceName,
            null,
            options =>
            {
                options.HttpMethod = HttpMethod.Get;
                options.RelativePath = $"api/todolist/{id}";
            });
        return View(value);
    }
   ```

#### <a name="option-3-call-a-downstream-web-api-without-the-helper-class"></a>Option 3 : Appeler une API web située en aval sans la classe d'assistance

Vous avez décidé d'acquérir un jeton manuellement à l'aide du service `ITokenAcquisition`, et vous devez maintenant utiliser le jeton. Le code suivant est la suite de l'exemple de code illustré dans [Une application web qui appelle des API web : Acquérir un jeton pour l’application](scenario-web-app-call-api-acquire-token.md). Le code est appelé dans les actions des contrôleurs d'applications web.

Une fois que vous avez acquis le jeton, utilisez-le comme un jeton de porteur pour appeler l'API située en aval, dans ce cas Microsoft Graph.

 ```csharp
public async Task<IActionResult> Profile()
{
 // Acquire the access token.
 string[] scopes = new string[]{"user.read"};
 string accessToken = await tokenAcquisition.GetAccessTokenForUserAsync(scopes);

 // Use the access token to call a protected web API.
 HttpClient client = new HttpClient();
 client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

  var response = await httpClient.GetAsync($"{webOptions.GraphApiUrl}/beta/me");

  if (response.StatusCode == HttpStatusCode.OK)
  {
   var content = await response.Content.ReadAsStringAsync();

   dynamic me = JsonConvert.DeserializeObject(content);
   ViewData["Me"] = me;
  }

  return View();
}
```
> [!NOTE]
> Vous pouvez utiliser le même principe pour appeler une API web.
>
> La plupart des API web Azure fournissent un kit de développement logiciel (SDK) qui simplifie l'appel de l'API, comme c'est le cas pour Microsoft Graph. Consultez [Créer une application web qui autorise l'accès au Stockage Blob avec Azure AD](../../storage/common/storage-auth-aad-app.md?tabs=dotnet&toc=%2fazure%2fstorage%2fblobs%2ftoc.json) pour accéder à un exemple d'application web utilisant Microsoft.Identity.Web et le SDK Stockage Azure.

# <a name="java"></a>[Java](#tab/java)

```Java
private String getUserInfoFromGraph(String accessToken) throws Exception {
    // Microsoft Graph user endpoint
    URL url = new URL("https://graph.microsoft.com/v1.0/me");

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
    return responseObject.toString();
}

```

# <a name="python"></a>[Python](#tab/python)

```Python
@app.route("/graphcall")
def graphcall():
    token = _get_token_from_cache(app_config.SCOPE)
    if not token:
        return redirect(url_for("login"))
    graph_data = requests.get(  # Use token to call downstream service.
        app_config.ENDPOINT,
        headers={'Authorization': 'Bearer ' + token['access_token']},
        ).json()
    return render_template('display.html', result=graph_data)
```

---

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Passer en production](scenario-web-app-call-api-production.md).