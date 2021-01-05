---
title: 'Tutoriel : Utiliser la configuration dynamique App Configuration dans ASP.NET Core'
titleSuffix: Azure App Configuration
description: Dans ce tutoriel, vous allez apprendre à mettre à jour dynamiquement les données de configuration pour les applications ASP.NET Core
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 09/1/2020
ms.author: alkemper
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 1fd495083f5f9be367dd0f125883b181e3bed27b
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96930549"
---
# <a name="tutorial-use-dynamic-configuration-in-an-aspnet-core-app"></a>Tutoriel : Utiliser la configuration dynamique dans une application ASP.NET Core

ASP.NET Core offre un système de configuration enfichable qui peut lire des données de configuration à partir de diverses sources. Il peut gérer les changements de manière dynamique sans entraîner le redémarrage d’une application. ASP.NET Core prend en charge la liaison des paramètres de configuration pour les classes .NET fortement typées. Il les injecte dans votre code à l’aide des différents modèles `IOptions<T>`. L’un de ces modèles, en particulier `IOptionsSnapshot<T>`, recharge automatiquement la configuration de l’application quand les données sous-jacentes changent. Vous pouvez injecter `IOptionsSnapshot<T>` dans des contrôleurs de votre application pour accéder à la configuration la plus récente stockée dans Azure App Configuration.

Vous pouvez également configurer la bibliothèque de client ASP.NET Core App Configuration pour actualiser dynamiquement un jeu de paramètres de configuration à l’aide d’un intergiciel. Les paramètres de configuration sont chaque fois mis à jour via le magasin de configuration aussi longtemps que l’application web reçoit des demandes.

App Configuration met automatiquement chaque paramètre en cache pour éviter un trop grand nombre d’appels au magasin de configuration. L’opération d’actualisation attend que la valeur mise en cache d’un paramètre ait expiré avant de mettre ce dernier à jour, même si sa valeur change dans le magasin de configuration. Le délai d’expiration du cache par défaut est de 30 secondes. Vous pouvez remplacer cette valeur, si nécessaire.

Ce tutoriel montre comment vous pouvez implémenter des mises à jour de la configuration dynamique dans votre code. Il s’appuie sur l’application web introduite dans les guides de démarrage rapide. Avant de continuer, terminez d’abord l’étape [Créer une application ASP.NET Core avec App Configuration](./quickstart-aspnet-core-app.md).

Vous pouvez utiliser l’éditeur de code de votre choix pour exécuter les étapes de ce tutoriel. [Visual Studio Code](https://code.visualstudio.com/) est une excellente option qui est disponible sur les plateformes Windows, macOS et Linux.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer votre application pour mettre à jour sa configuration en réponse aux changements survenant dans un magasin App Configuration.
> * Injecter la configuration la plus récente dans les contrôleurs de votre application.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, installez le [kit SDK .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Avant de continuer, terminez d’abord l’étape [Créer une application ASP.NET Core avec App Configuration](./quickstart-aspnet-core-app.md).

## <a name="add-a-sentinel-key"></a>Ajouter une clé Sentinel

Une *clé Sentinel* est une clé spéciale qui sert à signaler que la configuration a changé. Votre application supervise la clé Sentinel pour détecter les changements. Quand un changement est détecté, toutes les valeurs de configuration sont actualisées. Par rapport à l’approche qui consiste à rechercher les changements dans toutes les clés, celle-ci permet de limiter le nombre global de demandes que votre application adresse à App Configuration.

1. Sur le portail Azure, sélectionnez **Explorateur de configuration > Créer > Clé-valeur**.
1. Pour **Clé**, entrez *TestApp:Settings:Sentinel*. Pour **Valeur**, entrez 1. Laissez **Étiquette** et **Type de contenu** vides.
1. Sélectionnez **Appliquer**.

> [!NOTE]
> Si vous n’utilisez pas une clé Sentinel, vous devez inscrire manuellement chaque clé que vous souhaitez surveiller.

## <a name="reload-data-from-app-configuration"></a>Recharger des données à partir d’Azure App Configuration

1. Ajoutez une référence au package NuGet `Microsoft.Azure.AppConfiguration.AspNetCore` en exécutant la commande suivante :

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    ```

1. Ouvrez *Program.cs*, puis mettez à jour la méthode `CreateWebHostBuilder` pour ajouter la méthode `config.AddAzureAppConfiguration()`.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                           .ConfigureRefresh(refresh =>
                                {
                                    refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                           .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                });
                });
            })
            .UseStartup<Startup>();
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                    {
                        options.Connect(settings["ConnectionStrings:AppConfig"])
                               .ConfigureRefresh(refresh =>
                                    {
                                        refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                               .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                    });
                    });
                })
            .UseStartup<Startup>());
    ```
    ---

    La méthode `ConfigureRefresh` permet de spécifier les paramètres utilisés pour mettre à jour les données de configuration à l’aide du magasin App Configuration quand une opération d’actualisation est déclenchée. Le paramètre `refreshAll` de la méthode `Register` indique que toutes les valeurs de configuration doivent être actualisées si la clé Sentinel change.

    De même, la méthode `SetCacheExpiration` remplace le délai d’expiration par défaut du cache, la faisant passer de 30 secondes à 5 minutes. Cela permet de limiter le nombre de demandes adressées à App Configuration.

    > [!NOTE]
    > À des fins de test, vous pouvez abaisser le délai d’expiration du cache.

    Pour déclencher une opération d’actualisation, vous devez configurer un intergiciel (middleware) d’actualisation pour que l’application actualise les données de configuration en cas de changement. Vous verrez comment procéder dans une étape ultérieure.

2. Ajoutez un fichier *Settings.cs* qui définit et implémente une nouvelle classe `Settings`.

    ```csharp
    namespace TestAppConfig
    {
        public class Settings
        {
            public string BackgroundColor { get; set; }
            public long FontSize { get; set; }
            public string FontColor { get; set; }
            public string Message { get; set; }
        }
    }
    ```

3. Ouvrez *Startup.cs*, puis utilisez `IServiceCollection.Configure<T>` dans la méthode `ConfigureServices` pour lier les données de configuration à la classe `Settings`.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
    }
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddControllersWithViews();
    }
    ```
    ---
    > [!Tip]
    > Pour en savoir plus sur le modèle d’options pour la lecture des valeurs de configuration, consultez [Modèles d’options dans ASP.NET Core](/aspnet/core/fundamentals/configuration/options?view=aspnetcore-3.1).

4. Mettez à jour la méthode `Configure`, en ajoutant l’intergiciel (middleware) `UseAzureAppConfiguration` afin de permettre la mise à jour des paramètres de configuration inscrits pour être actualisés pendant que l’application web ASP.NET Core continue de recevoir des requêtes.


    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseAzureAppConfiguration();

        services.Configure<CookiePolicyOptions>(options =>
        {
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        app.UseMvc();
    }
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            // Add the following line:
            app.UseAzureAppConfiguration();

            app.UseHttpsRedirection();
            
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
    }
    ```
    ---
    
    L’intergiciel utilise la configuration d’actualisation spécifiée dans la méthode `AddAzureAppConfiguration` de `Program.cs` afin de déclencher une actualisation pour chaque demande reçue par l’application web ASP.NET Core. Pour chaque demande, une opération d’actualisation est déclenchée et la bibliothèque de client vérifie si la valeur mise en cache pour les paramètres de configuration inscrits a expiré. Si elle a expiré, elle est actualisée.

    > [!NOTE]
    > Pour vous assurer que la configuration est actualisée, ajoutez le middleware dès qu’il convient à votre pipeline de demande afin qu’il ne soit pas court-circuité par un autre middleware dans votre application.

## <a name="use-the-latest-configuration-data"></a>Utiliser les données de configuration les plus récentes

1. Ouvrez *HomeController.cs* dans le répertoire Controllers et ajoutez une référence au package `Microsoft.Extensions.Options`.

    ```csharp
    using Microsoft.Extensions.Options;
    ```

2. Mettez à jour la classe `HomeController` pour recevoir `Settings` par le biais de l’injection de dépendance, puis utilisez ses valeurs.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        public HomeController(IOptionsSnapshot<Settings> settings)
        {
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }
    }
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger, IOptionsSnapshot<Settings> settings)
        {
            _logger = logger;
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }

        // ...
    }
    ```
    ---



3. Ouvrez *Index.cshtml* dans le répertoire Views > Home, puis remplacez son contenu par le script suivant :

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <style>
        body {
            background-color: @ViewData["BackgroundColor"]
        }
        h1 {
            color: @ViewData["FontColor"];
            font-size: @ViewData["FontSize"]px;
        }
    </style>
    <head>
        <title>Index View</title>
    </head>
    <body>
        <h1>@ViewData["Message"]</h1>
    </body>
    </html>
    ```

## <a name="build-and-run-the-app-locally"></a>Générer et exécuter l’application localement

1. Pour générer l’application à l’aide de l’interface CLI .NET Core, exécutez la commande suivante dans l’interpréteur de commandes :

    ```console
        dotnet build
    ```

1. Une fois la génération correctement terminée, exécutez la commande suivante pour exécuter l’application web localement :

    ```console
        dotnet run
    ```

1. Ouvrez une fenêtre de navigateur, puis accédez à l’URL figurant dans la sortie de `dotnet run`.

    ![Lancement local de l’application du démarrage rapide](./media/quickstarts/aspnet-core-app-launch-local-before.png)

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Toutes les ressources**, puis sélectionnez l’instance du magasin App Configuration que vous avez créée dans le guide de démarrage rapide.

1. Sélectionnez **Explorateur de configuration**, puis mettez à jour les valeurs des clés suivantes :

    | Clé | Valeur |
    |---|---|
    | TestApp:Settings:BackgroundColor | green |
    | TestApp:Settings:FontColor | lightGray |
    | TestApp:Settings:Message | Données provenant d’Azure App Configuration, avec maintenant des mises à jour automatiques ! |
    | TestApp:Settings:Sentinel | 2 |

1. Actualisez la page de navigateur pour afficher les nouveaux paramètres de configuration. Vous devrez peut-être effectuer plusieurs actualisations pour faire apparaître les changements.

    ![Lancement local de l’application mise à jour du démarrage rapide](./media/quickstarts/aspnet-core-app-launch-local-after.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez permis à votre application web ASP.NET Core d’actualiser dynamiquement les paramètres de configuration à partir d’App Configuration. Pour savoir comment utiliser une identité managée Azure de façon à simplifier l’accès à App Configuration, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Intégration des identités managées](./howto-integrate-azure-managed-service-identity.md)
