---
title: Tutoriel sur l’utilisation des indicateurs de fonctionnalités dans une application .NET Core | Microsoft Docs
description: Dans ce tutoriel, vous allez découvrir comment implémenter des indicateurs de fonctionnalités dans des applications .NET Core.
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 09/17/2020
ms.author: alkemper
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 8c0dd9713c673ad676058acc7dbbb3cb5a65362e
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929189"
---
# <a name="tutorial-use-feature-flags-in-an-aspnet-core-app"></a>Tutoriel : Utiliser des indicateurs de fonctionnalités dans une application ASP.NET Core

Les bibliothèques de gestion des fonctionnalités .NET Core fournissent une prise en charge idiomatique de l’implémentation des indicateurs de fonctionnalités dans une application .NET ou ASP.NET Core. Ces bibliothèques vous permettent d’ajouter de manière déclarative des indicateurs de fonctionnalités à votre code afin de vous éviter d’écrire manuellement toutes les instructions `if` correspondantes.

Les bibliothèques de gestion des fonctionnalités gèrent également les cycles de vie des indicateurs de fonctionnalités en arrière-plan. Par exemple, elles actualisent et mettent en cache l’état des indicateurs ou garantissent qu’un état d’indicateur est immuable pendant un appel de demande. De plus, la bibliothèque ASP.NET Core offre des intégrations prêtes à l’emploi, notamment des intergiciels (middleware), des routes, des vues et des actions de contrôleur MVC.

Le [guide de démarrage rapide Ajouter des indicateurs de fonctionnalités dans une application ASP.NET Core](./quickstart-feature-flag-aspnet-core.md) montre plusieurs façons d’ajouter des indicateurs de fonctionnalités à une application ASP.NET Core. Ce tutoriel explique plus en détails ces différentes méthodes. Pour obtenir une référence complète, consultez la [documentation sur la gestion des fonctionnalités ASP.NET Core](/dotnet/api/microsoft.featuremanagement).

Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Ajouter des indicateurs de fonctionnalités dans des parties clés de votre application afin de contrôler la disponibilité des fonctionnalités
> * Effectuer une intégration à App Configuration quand vous l’utilisez pour gérer les indicateurs de fonctionnalités.

## <a name="set-up-feature-management"></a>Configurer la gestion des fonctionnalités

Ajoutez une référence aux packages NuGet `Microsoft.FeatureManagement.AspNetCore` et `Microsoft.FeatureManagement` pour utiliser le gestionnaire de fonctionnalités .NET Core.
    
Le gestionnaire de fonctionnalités de .NET Core `IFeatureManager` obtient les indicateurs de fonctionnalités à partir du système de configuration natif du framework. En conséquence, vous pouvez définir les indicateurs de fonctionnalités de votre application à l’aide de n’importe quelle source de configuration prise en charge par .NET Core, notamment le fichier *appsettings.json* local ou des variables d’environnement. `IFeatureManager` s’appuie sur l’injection de dépendances .NET Core. Vous pouvez inscrire les services de gestion des fonctionnalités à l’aide de conventions standard :

```csharp
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement();
    }
}
```

Par défaut, le gestionnaire de fonctionnalités récupère les indicateurs de fonctionnalités à partir de la section `"FeatureManagement"` des données de configuration .NET Core. L’exemple suivant fait en sorte qu’il lise à partir d’une autre section nommée `"MyFeatureFlags"` à la place :

```csharp
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement(options =>
        {
                options.UseConfiguration(Configuration.GetSection("MyFeatureFlags"));
        });
    }
}
```

Si vous utilisez des filtres dans vos indicateurs de fonctionnalités, vous devez inclure une bibliothèque supplémentaire et l’inscrire. L’exemple suivant montre comment utiliser un filtre de fonctionnalité intégré nommé `PercentageFilter` :

```csharp
using Microsoft.FeatureManagement;
using Microsoft.FeatureManagement.FeatureFilters;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement()
                .AddFeatureFilter<PercentageFilter>();
    }
}
```

Nous vous recommandons de conserver les indicateurs de fonctionnalités en dehors de l’application et de les gérer séparément. Cela vous permet de modifier l’état des indicateurs à tout moment, et d’appliquer immédiatement ces changements dans l’application. Le service App Configuration fournit un emplacement centralisé pour organiser et contrôler tous vos indicateurs de fonctionnalités par le biais d’une interface utilisateur de portail dédiée. Il remet également les indicateurs à votre application directement par le biais de ses bibliothèques clientes .NET Core.

Le moyen le plus simple de connecter votre application ASP.NET Core à App Configuration est de passer par l’objet `Microsoft.Azure.AppConfiguration.AspNetCore` du fournisseur de configuration. Suivez ces étapes pour utiliser ce package NuGet.

1. Ouvrez le fichier *Program.cs* et ajoutez le code suivant.

   ```csharp
   using Microsoft.Extensions.Configuration.AzureAppConfiguration;

   public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
       WebHost.CreateDefaultBuilder(args)
              .ConfigureAppConfiguration((hostingContext, config) => {
                  var settings = config.Build();
                  config.AddAzureAppConfiguration(options => {
                      options.Connect(settings["ConnectionStrings:AppConfig"])
                             .UseFeatureFlags();
                   });
              })
              .UseStartup<Startup>();
   ```

2. Ouvrez *Startup.cs* et mettez à jour la méthode `Configure` pour ajouter l’intergiciel (middleware) intégré appelé `UseAzureAppConfiguration`. Ce middleware permet l’actualisation des valeurs d’indicateurs de fonctionnalités à intervalles réguliers pendant que l’application web ASP.NET continue de recevoir des requêtes.

   ```csharp
   public void Configure(IApplicationBuilder app, IHostingEnvironment env)
   {
       app.UseAzureAppConfiguration();
       app.UseMvc();
   }
   ```

Les valeurs des indicateurs de fonctionnalités sont supposées changer au fil du temps. Par défaut, les valeurs d’indicateur de fonctionnalité sont mises en cache pendant une période de 30 secondes. Une opération d’actualisation déclenchée quand le middleware reçoit une demande ne met donc à jour la valeur qu’après l’expiration de la valeur mise en cache. Le code suivant montre comment faire passer l’heure d’expiration du cache ou l’intervalle d’interrogation à 5 minutes dans l’appel `options.UseFeatureFlags()`.

```csharp
config.AddAzureAppConfiguration(options => {
    options.Connect(settings["ConnectionStrings:AppConfig"])
           .UseFeatureFlags(featureFlagOptions => {
                featureFlagOptions.CacheExpirationTime = TimeSpan.FromMinutes(5);
           });
});
```

## <a name="feature-flag-declaration"></a>Déclaration d’indicateur de fonctionnalité

Chaque indicateur de fonctionnalité comporte deux parties : un nom et une liste d’un ou plusieurs filtres qui servent à évaluer si l’état d’une fonctionnalité est *on* (autrement dit, quand sa valeur est `True`). Un filtre définit un cas d’usage pour le moment où une fonctionnalité doit être activée.

Si un indicateur de fonctionnalité a plusieurs filtres, la liste des filtres est parcourue dans l’ordre jusqu’à ce que l’un d’eux détermine que la fonctionnalité doit être activée. À ce stade, l’indicateur de fonctionnalité est *on* (activée) et les résultats du filtrage restants sont ignorés. Si aucun filtre n’indique que la fonctionnalité doit être activée, l’indicateur de fonctionnalité est *off* (désactivée).

Le gestionnaire de fonctionnalités prend en charge *appsettings.json* comme source de configuration pour les indicateurs de fonctionnalités. L’exemple suivant montre comment configurer des indicateurs de fonctionnalités dans un fichier JSON :

```JSON
"FeatureManagement": {
    "FeatureA": true, // Feature flag set to on
    "FeatureB": false, // Feature flag set to off
    "FeatureC": {
        "EnabledFor": [
            {
                "Name": "Percentage",
                "Parameters": {
                    "Value": 50
                }
            }
        ]
    }
}
```

Par convention, la section `FeatureManagement` de ce document JSON est utilisée pour les paramètres des indicateurs de fonctionnalités. L’exemple précédent montre trois indicateurs de fonctionnalités avec leurs filtres définis dans la propriété `EnabledFor` :

* `FeatureA` est défini sur *on*.
* `FeatureB` est défini sur *off*.
* `FeatureC` spécifie un filtre nommé `Percentage` avec une propriété `Parameters`. `Percentage` est un filtre configurable. Dans cet exemple, `Percentage` spécifie une probabilité de 50 % que l’indicateur `FeatureC` soit défini sur *on*.

## <a name="feature-flag-references"></a>Références des indicateurs de fonctionnalités

Pour pouvoir aisément référencer des indicateurs de fonctionnalités dans du code, vous devez les définir en tant que variables `enum` :

```csharp
public enum MyFeatureFlags
{
    FeatureA,
    FeatureB,
    FeatureC
}
```

## <a name="feature-flag-checks"></a>Vérifications des indicateurs de fonctionnalités

Le modèle de gestion des fonctionnalités de base consiste à d’abord vérifier si un indicateur de fonctionnalité est défini sur *on*. Si c’est le cas, le gestionnaire de fonctionnalités exécute les actions que la fonctionnalité contient. Par exemple :

```csharp
IFeatureManager featureManager;
...
if (await featureManager.IsEnabledAsync(nameof(MyFeatureFlags.FeatureA)))
{
    // Run the following code
}
```

## <a name="dependency-injection"></a>Injection de dépendances

Dans ASP.NET Core MVC, vous pouvez accéder au gestionnaire de fonctionnalités `IFeatureManager` par le biais de l’injection de dépendances :

```csharp
public class HomeController : Controller
{
    private readonly IFeatureManager _featureManager;

    public HomeController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }
}
```

## <a name="controller-actions"></a>Actions de contrôleur

Dans les contrôleurs MVC, vous pouvez utiliser un attribut `FeatureGate` pour déterminer si l’ensemble d’une classe de contrôleur ou une action spécifique est activée. Le contrôleur `HomeController` suivant exige que `FeatureA` soit défini sur *on* avant l’exécution de toute action que contient la classe de contrôleur :

```csharp
using Microsoft.FeatureManagement.Mvc;

[FeatureGate(MyFeatureFlags.FeatureA)]
public class HomeController : Controller
{
    ...
}
```

L’action `Index` suivante exige que `FeatureA` soit défini sur *on* pour pouvoir être exécutée :

```csharp
using Microsoft.FeatureManagement.Mvc;

[FeatureGate(MyFeatureFlags.FeatureA)]
public IActionResult Index()
{
    return View();
}
```

Quand une action ou un contrôleur MVC est bloqué car l’indicateur de fonctionnalité de contrôle est défini sur *off*, une interface `IDisabledFeaturesHandler` inscrite est appelée. L’interface `IDisabledFeaturesHandler` par défaut retourne un code d’état 404 au client sans corps de réponse.

## <a name="mvc-views"></a>Vues MVC

Ouvrez *_ViewImports.cshtml* dans le répertoire *Views*, puis ajoutez le tag helper du gestionnaire de fonctionnalités :

```html
@addTagHelper *, Microsoft.FeatureManagement.AspNetCore
```

Dans les vues MVC, vous pouvez utiliser une balise `<feature>` pour restituer le contenu si un indicateur de fonctionnalité est activé :

```html
<feature name="FeatureA">
    <p>This can only be seen if 'FeatureA' is enabled.</p>
</feature>
```

Pour afficher un contenu alternatif quand les conditions requises ne sont pas remplies, l’attribut `negate` peut être utilisé.

```html
<feature name="FeatureA" negate="true">
    <p>This will be shown if 'FeatureA' is disabled.</p>
</feature>
```

La balise de fonctionnalité `<feature>` peut également être utilisée pour afficher le contenu si certaines ou l’ensemble des fonctionnalités dans une liste sont activées.

```html
<feature name="FeatureA, FeatureB" requirement="All">
    <p>This can only be seen if 'FeatureA' and 'FeatureB' are enabled.</p>
</feature>
<feature name="FeatureA, FeatureB" requirement="Any">
    <p>This can be seen if 'FeatureA', 'FeatureB', or both are enabled.</p>
</feature>
```

## <a name="mvc-filters"></a>Filtres MVC

Vous pouvez configurer des filtres MVC de sorte que leur activation soit basée sur l’état d’un indicateur de fonctionnalité. Le code suivant ajoute un filtre MVC nommé `SomeMvcFilter`. Ce filtre est déclenché dans le pipeline MVC seulement si `FeatureA` est activé. Cette fonctionnalité est limitée à `IAsyncActionFilter`. 

```csharp
using Microsoft.FeatureManagement.FeatureFilters;

IConfiguration Configuration { get; set;}

public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options => {
        options.Filters.AddForFeature<SomeMvcFilter>(nameof(MyFeatureFlags.FeatureA));
    });
}
```

## <a name="middleware"></a>Middlewares

Vous pouvez également utiliser des indicateurs de fonctionnalités pour ajouter de manière conditionnelle des intergiciels (middlewares) et des branches d’application. Le code suivant insère un composant d’intergiciel dans le pipeline de requête uniquement quand `FeatureA` est activé :

```csharp
app.UseMiddlewareForFeature<ThirdPartyMiddleware>(nameof(MyFeatureFlags.FeatureA));
```

Ce code repose sur la capacité plus générique qui consiste à créer une branche de l’ensemble de l’application entière en fonction d’un indicateur de fonctionnalité :

```csharp
app.UseForFeature(featureName, appBuilder => {
    appBuilder.UseMiddleware<T>();
});
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez découvert comment implémenter des indicateurs de fonctionnalités dans votre application ASP.NET Core en utilisant les bibliothèques `Microsoft.FeatureManagement`. Pour plus d’informations sur la prise en charge de la gestion des fonctionnalités dans ASP.NET Core et App Configuration, consultez les ressources suivantes :

* [Exemple de code d’indicateur de fonctionnalité ASP.NET Core](./quickstart-feature-flag-aspnet-core.md)
* [Documentation de Microsoft.FeatureManagement](/dotnet/api/microsoft.featuremanagement)
* [Gérer les indicateurs de fonctionnalité](./manage-feature-flags.md)