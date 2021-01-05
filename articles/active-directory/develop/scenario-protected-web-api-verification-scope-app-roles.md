---
title: Vérifier les étendues et les rôles d’application avec une API web protégée | Azure
titleSuffix: Microsoft identity platform
description: Vérifiez que l’API est appelée uniquement par les applications pour le compte des utilisateurs qui ont les étendues adéquates et par les applications de démon ayant les rôles d’application appropriés.
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
ms.openlocfilehash: 6d737f107e7a9b1476fdf86ac0320931ea137671
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94442903"
---
# <a name="protected-web-api-verify-scopes-and-app-roles"></a>API web protégée : Vérifier les étendues et les rôles d’application

Cet article décrit comment ajouter une autorisation à votre API web. Cette protection garantit que l’API est appelée uniquement par :

- Des applications pour le compte d’utilisateurs qui ont des étendues adaptées.
- Des applications de démon qui ont les rôles d’application appropriés.

> [!NOTE]
> Les extraits de code de cet article proviennent des exemples de code suivants sur GitHub :
>
> - [Tutoriel incrémentiel sur l’API web ASP.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/blob/master/1.%20Desktop%20app%20calls%20Web%20API/TodoListService/Controllers/TodoListController.cs)
> - [Exemple d’API web ASP.NET](https://github.com/Azure-Samples/ms-identity-aspnet-webapi-onbehalfof/blob/master/TodoListService/Controllers/TodoListController.cs)

Pour protéger une API web ASP.NET ou ASP.NET Core, vous devez ajouter l’attribut `[Authorize]` à l’un des éléments suivants :

- Le contrôleur proprement dit, si vous voulez que toutes les actions de contrôleur soient protégées
- L’action de contrôleur individuelle pour votre API

```csharp
    [Authorize]
    public class TodoListController : Controller
    {
     ...
    }
```

Mais cette protection n’est pas suffisante. Elle garantit uniquement qu’ASP.NET et ASP.NET Core valident le jeton. Votre API doit vérifier que le jeton utilisé pour appeler l’API est demandée avec les revendications attendues. Ces revendications doivent notamment être vérifiées :

- Les *étendues* si l’API est appelée pour le compte d’un utilisateur.
- Les *rôles d’application* si l’API peut être appelée à partir d’une application démon.

## <a name="verify-scopes-in-apis-called-on-behalf-of-users"></a>Vérifier les étendues dans les API appelées pour le compte d’utilisateurs

Si une application cliente appelle votre API pour le compte d’un utilisateur, l’API doit demander un jeton du porteur ayant des étendues spécifiques pour elle. Pour plus d’informations, consultez [Configuration de code | Jeton du porteur](scenario-protected-web-api-app-configuration.md#bearer-token).

### <a name="net-core"></a>.NET Core

#### <a name="verify-the-scopes-on-each-controller-action"></a>Vérifier les étendues de chaque action de contrôleur

```csharp
[Authorize]
public class TodoListController : Controller
{
    /// <summary>
    /// The web API will accept only tokens 1) for users, 2) that have the `access_as_user` scope for
    /// this API.
    /// </summary>
    static readonly string[] scopeRequiredByApi = new string[] { "access_as_user" };

    // GET: api/values
    [HttpGet]
    public IEnumerable<TodoItem> Get()
    {
         HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);
        // Do the work and return the result.
        // ...
    }
...
}
```

La méthode `VerifyUserHasAnyAcceptedScope` effectue des étapes semblables aux suivantes :

- Vérifier qu’il existe une revendication nommée `http://schemas.microsoft.com/identity/claims/scope` ou `scp`.
- Vérifier que la revendication a une valeur qui contient l’étendue attendue par l’API.


#### <a name="verify-the-scopes-more-globally"></a>Vérifier les étendues plus globalement

La définition d’étendues précises pour votre API web et la vérification des étendues dans chaque action de contrôleur est l’approche recommandée. Toutefois, il est également possible de vérifier les étendues au niveau de l’application ou d’un contrôleur à l’aide d’ASP.NET Core. Pour plus d’informations, consultez [Autorisation basée sur les revendications](/aspnet/core/security/authorization/claims) dans la documentation d’ASP.NET Core.

### <a name="net-mvc"></a>.NET MVC

Pour ASP.NET, remplacez simplement `HttpContext.User` par `ClaimsPrincipal.Current` et remplacez le type de revendication `"http://schemas.microsoft.com/identity/claims/scope"` par `"scp"`. Consultez également l’extrait de code présenté plus loin dans cet article.

## <a name="verify-app-roles-in-apis-called-by-daemon-apps"></a>Vérifier les rôles d’application dans les API appelées par des applications démon

Si votre API web est appelée par une [application de démon](scenario-daemon-overview.md), cette application doit exiger une autorisation d’application à votre API web. Comme indiqué dans [Exposition des autorisations d’application (rôles d’application)](./scenario-protected-web-api-app-registration.md#exposing-application-permissions-app-roles), votre API expose de telles autorisations. Le rôle d’application `access_as_application` en est un exemple.

Vous devez maintenant demander à votre API de vérifier que le jeton qu’elle reçoit contient la revendication `roles` et que cette revendication a la valeur attendue. Le code de vérification est similaire au code qui vérifie les autorisations déléguées, excepté que votre action de contrôleur teste les rôles au lieu des étendues :

### <a name="aspnet-core"></a>ASP.NET Core

```csharp
[Authorize]
public class TodoListController : ApiController
{
    public IEnumerable<TodoItem> Get()
    {
        HttpContext.ValidateAppRole("access_as_application");
        ...
    }
```

La méthode `ValidateAppRole` est définie dans Microsoft.Identity.Web au sein de [RolesRequiredHttpContextExtensions.cs](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/Resource/RolesRequiredHttpContextExtensions.cs#L28).

### <a name="aspnet-mvc"></a>ASP.NET MVC

```csharp
private void ValidateAppRole(string appRole)
{
    //
    // The `role` claim tells you what permissions the client application has in the service.
    // In this case, we look for a `role` value of `access_as_application`.
    //
    Claim roleClaim = ClaimsPrincipal.Current.FindFirst("roles");
    if (roleClaim == null || !roleClaim.Value.Split(' ').Contains(appRole))
    {
        throw new HttpResponseException(new HttpResponseMessage
        { StatusCode = HttpStatusCode.Unauthorized,
            ReasonPhrase = $"The 'roles' claim does not contain '{appRole}' or was not found"
        });
    }
}
}
```

### <a name="accepting-app-only-tokens-if-the-web-api-should-be-called-only-by-daemon-apps"></a>Acceptation de jetons d’application uniquement si l’API web doit être appelée uniquement par des applications de démon

Les utilisateurs peuvent également utiliser des revendications de rôles dans des modèles d’affectation d’utilisateurs, comme indiqué dans [Guide pratique pour ajouter des rôles d’application dans votre application et les recevoir dans le jeton](howto-add-app-roles-in-azure-ad-apps.md). Si les rôles peuvent être attribués aux deux, vérifier les rôles permet aux applications de se connecter en tant qu’utilisateurs et aux utilisateurs de se connecter en tant qu’applications. Nous vous recommandons de déclarer des rôles différents pour les utilisateurs et les applications afin d’éviter cette confusion.

Si vous voulez que seules les applications démon appellent votre API web, ajoutez la condition indiquant que le jeton est un jeton d’application uniquement quand vous validez le rôle d’application.

```csharp
string oid = ClaimsPrincipal.Current.FindFirst("oid")?.Value;
string sub = ClaimsPrincipal.Current.FindFirst("sub")?.Value;
bool isAppOnlyToken = oid == sub;
```

La vérification de la condition inverse autorise uniquement les applications qui connectent un utilisateur à appeler votre API.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Passer en production](scenario-protected-web-api-production.md).
