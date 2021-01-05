---
title: API web appelant des API web - Plateforme d’identités Microsoft | Azure
description: Découvrez comment générer une API web qui appelle des API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/26/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: b8eb737752741dc55da24ad6b605b081053b8a11
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443699"
---
# <a name="a-web-api-that-calls-web-apis-call-an-api"></a>Une API web qui appelle des API web : Appeler une API

Après avoir obtenu un jeton, vous pouvez appeler une API web protégée. Vous appelez généralement les API située en aval à partir du contrôleur ou de pages de votre API web.

## <a name="controller-code"></a>Code de contrôle

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Lorsque vous utilisez *Microsoft.Identity.Web*, vous avez trois scénarios d’utilisation :

- [Option n°1 : Appeler Microsoft Graph à l’aide du kit de développement logiciel (SDK) Microsoft Graph](#option-1-call-microsoft-graph-with-the-sdk)
- [Option n°2 : Appeler une API web située en aval avec la classe d’assistance](#option-2-call-a-downstream-web-api-with-the-helper-class)
- [Option 3 : Appeler une API web située en aval sans la classe d’assistance](#option-3-call-a-downstream-web-api-without-the-helper-class)

#### <a name="option-1-call-microsoft-graph-with-the-sdk"></a>Option 1 : Appeler Microsoft Graph à l’aide du SDK

Dans ce scénario, vous avez ajouté `.AddMicrosoftGraph()` dans le fichier *Startup.cs* comme spécifié dans [Configuration de code](scenario-web-api-call-api-app-configuration.md#option-1-call-microsoft-graph), et vous pouvez injecter directement le `GraphServiceClient` dans votre contrôleur ou constructeur de page pour l’utiliser dans les actions. L’exemple de page Razor suivant affiche la photo de l’utilisateur connecté.

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

#### <a name="option-2-call-a-downstream-web-api-with-the-helper-class"></a>Option n°2 : Appeler une API web située en aval avec la classe d’assistance

Vous avez ici ajouté `.AddDownstreamWebApi()` dans le fichier *Startup.cs* comme spécifié dans [Configuration de code](scenario-web-api-call-api-app-configuration.md#option-2-call-a-downstream-web-api-other-than-microsoft-graph), et vous pouvez injecter directement un service `IDownstreamWebApi` dans votre contrôleur ou constructeur de page pour l’utiliser dans les actions :

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
```

La méthode `CallWebApiForUserAsync` possède également des remplacements génériques fortement typés qui vous permettent de recevoir directement un objet. Par exemple, la méthode suivante a reçu une instance `Todo`, qui est une représentation fortement typée du code JSON renvoyé par l’API web.

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

#### <a name="option-3-call-a-downstream-web-api-without-the-helper-class"></a>Option 3 : Appeler une API web située en aval sans la classe d’assistance

Si vous avez décidé d’acquérir un jeton manuellement à l’aide du service `ITokenAcquisition`, vous devez maintenant utiliser le jeton. Dans ce cas, le code suivant est la suite de l’exemple de code illustré dans [Une API web qui appelle des API web : Acquérir un jeton pour l’application](scenario-web-api-call-api-acquire-token.md). Le code est appelé dans les actions des contrôleurs d’API. Il appelle une API en aval nommée *todolist*.

 Une fois que vous avez acquis le jeton, utilisez-le comme un jeton du porteur pour appeler l’API en aval.

```csharp
 private async Task CallTodoListService(string accessToken)
 {
  // After the token has been returned by Microsoft.Identity.Web, add it to the HTTP authorization header before making the call to access the todolist service.
 _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

 // Call the todolist service.
 HttpResponseMessage response = await _httpClient.GetAsync(TodoListBaseAddress + "/api/todolist");
 // ...
 }
 ```

# <a name="java"></a>[Java](#tab/java)

Le code suivant poursuit l’exemple de code illustré dans [Une API web qui appelle des API web : Acquérir un jeton pour l’application](scenario-web-api-call-api-acquire-token.md). Le code est appelé dans les actions des contrôleurs d’API. Il appelle l’API MS Graph en aval.

Une fois que vous avez acquis le jeton, utilisez-le comme un jeton du porteur pour appeler l’API en aval.

```Java
private String callMicrosoftGraphMeEndpoint(String accessToken){
    RestTemplate restTemplate = new RestTemplate();

    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);

    headers.set("Authorization", "Bearer " + accessToken);

    HttpEntity<String> entity = new HttpEntity<>(null, headers);

    String result = restTemplate.exchange("https://graph.microsoft.com/v1.0/me", HttpMethod.GET,
            entity, String.class).getBody();

    return result;
}
```

# <a name="python"></a>[Python](#tab/python)
Aucun exemple illustrant ce flux avec MSAL Python n’est encore disponible.

---

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Passer en production](scenario-web-api-call-api-production.md).
