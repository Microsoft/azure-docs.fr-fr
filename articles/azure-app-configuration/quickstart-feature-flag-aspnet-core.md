---
title: Démarrage rapide pour l’ajout d’indicateurs de fonctionnalité à ASP.NET Core
description: Ajouter des indicateurs de fonctionnalité à des applications ASP.NET Core et les gérer dans Azure App Configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.custom: devx-track-csharp
ms.topic: quickstart
ms.date: 09/28/2020
ms.author: alkemper
ms.openlocfilehash: d465f3c44ede8b4df56ef0da08c5bbbcd477d93f
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932136"
---
# <a name="quickstart-add-feature-flags-to-an-aspnet-core-app"></a>Démarrage rapide : Ajouter des indicateurs de fonctionnalités dans une application ASP.NET Core

Dans ce guide de démarrage rapide, vous allez créer une implémentation de bout en bout de la gestion des fonctionnalités dans une application ASP.NET Core à l'aide d'Azure App Configuration. Vous allez utiliser le service App Configuration pour stocker de façon centralisée tous vos indicateurs de fonctionnalité et contrôler leur état. 

Les bibliothèques de gestion des fonctionnalités .NET Core étendent le framework avec une prise en charge complète des indicateurs de fonctionnalités. Ces bibliothèques sont basées sur le système de configuration .NET Core. Elles s’intègrent de manière fluide à App Configuration par le biais de son fournisseur de configuration .NET Core.

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/dotnet)
* [Kit de développement logiciel (SDK) .NET Core](https://dotnet.microsoft.com/download)

## <a name="create-an-app-configuration-store"></a>Créer un magasin App Configuration

[!INCLUDE[Azure App Configuration resource creation steps](../../includes/azure-app-configuration-create.md)]

8. Sélectionnez **Opérations** > **Gestionnaire de fonctionnalités** > **Ajouter** pour ajouter un indicateur de fonctionnalité appelé *Beta*.

    > [!div class="mx-imgBorder"]
    > ![Activer l’indicateur de fonctionnalité nommé Beta](media/add-beta-feature-flag.png)

    Laissez le champ **Étiquette** vide pour le moment. Sélectionnez **Appliquer** pour enregistrer le nouvel indicateur de fonctionnalité.

## <a name="create-an-aspnet-core-web-app"></a>Créez une application web ASP.NET Core

Utilisez l'[interface de ligne de commande (CLI) .NET Core](/dotnet/core/tools) pour créer un projet de MVC ASP.NET Core. Par rapport à Visual Studio, l’interface CLI .NET Core offre l’avantage d’être disponible sur les plateformes Windows, macOS et Linux.

Exécutez la commande suivante pour créer un projet MVC ASP.NET Core dans un nouveau dossier *TestFeatureFlags* :

```dotnetcli
dotnet new mvc --no-https --output TestFeatureFlags
```

[!INCLUDE[Add Secret Manager support to an ASP.NET Core project](../../includes/azure-app-configuration-add-secret-manager.md)]

## <a name="connect-to-an-app-configuration-store"></a>Se connecter à un magasin App Configuration

1. Installez les packages NuGet [Microsoft.Azure.AppConfiguration.AspNetCore](https://www.nuget.org/packages/Microsoft.Azure.AppConfiguration.AspNetCore) et [Microsoft.FeatureManagement.AspNetCore](https://www.nuget.org/packages/Microsoft.FeatureManagement.AspNetCore) en exécutant les commandes suivantes :

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    ```

    ```dotnetcli
    dotnet add package Microsoft.FeatureManagement.AspNetCore
    ```

1. Exécutez la commande suivante dans le répertoire où se trouve le fichier *.csproj*. La commande utilise Secret Manager pour stocker un secret nommé `ConnectionStrings:AppConfig`, qui stocke la chaîne de connexion de votre magasin App Configuration. Remplacez l'espace réservé `<your_connection_string>` par la chaîne de connexion de votre magasin App Configuration. La chaîne de connexion se trouve sous **Clés d’accès** dans le portail Azure.

    ```dotnetcli
    dotnet user-secrets set ConnectionStrings:AppConfig "<your_connection_string>"
    ```

    Secret Manager est utilisé uniquement pour tester l’application web localement. Une fois l'application déployée sur [Azure App Service](https://azure.microsoft.com/services/app-service/web), utilisez le paramètre d'application **Chaînes de connexion** d'App Service plutôt que Secret Manager pour stocker la chaîne de connexion.

    Accédez à ce secret à l'aide de l'API de configuration .NET Core. Avec l'API de configuration, un signe deux-points (`:`) fonctionne dans le nom de configuration sur toutes les plateformes prises en charge. Pour plus d'informations, consultez [Clés et valeurs de configuration](/aspnet/core/fundamentals/configuration#configuration-keys-and-values).

1. Dans *Program.cs*, mettez à jour la méthode `CreateWebHostBuilder` pour utiliser App Configuration en appelant la méthode `AddAzureAppConfiguration`.

    > [!IMPORTANT]
    > `CreateHostBuilder` remplace `CreateWebHostBuilder` dans .NET Core 3.x. Sélectionnez la syntaxe appropriée en fonction de votre environnement.

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration(config =>
                {
                    var settings = config.Build();
                    var connection = settings.GetConnectionString("AppConfig");
                    config.AddAzureAppConfiguration(options =>
                        options.Connect(connection).UseFeatureFlags());
                }).UseStartup<Startup>());
    ```

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
               .ConfigureAppConfiguration(config =>
               {
                   var settings = config.Build();
                   var connection = settings.GetConnectionString("AppConfig");
                   config.AddAzureAppConfiguration(options =>
                       options.Connect(connection).UseFeatureFlags());
               }).UseStartup<Startup>();
    ```

    ---

    Avec la modification précédente, le [fournisseur de configuration d'App Configuration](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration) a été inscrit auprès de l'API de configuration .NET Core.

1. Dans *Startup.cs*, ajoutez une référence au gestionnaire de fonctionnalités .NET Core :

    ```csharp
    using Microsoft.FeatureManagement;
    ```

1. Mettez à jour la méthode `Startup.ConfigureServices` pour ajouter la prise en charge de l’indicateur de fonctionnalité en appelant la méthode `AddFeatureManagement`. Si vous le souhaitez, vous pouvez inclure un filtre à utiliser avec les indicateurs de fonctionnalité en appelant `AddFeatureFilter<FilterType>()` :

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp    
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews();
        services.AddFeatureManagement();
    }
    ```

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc()
            .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
        services.AddFeatureManagement();
    }
    ```

    ---

1. Ajoutez un fichier *MyFeatureFlags.cs* au répertoire racine du projet avec le code suivant :

    ```csharp
    namespace TestFeatureFlags
    {
        public enum MyFeatureFlags
        {
            Beta
        }
    }
    ```

1. Ajoutez un fichier *BetaController.cs* au répertoire *Controllers* avec le code suivant :

    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.FeatureManagement;
    using Microsoft.FeatureManagement.Mvc;

    namespace TestFeatureFlags.Controllers
    {
        public class BetaController: Controller
        {
            private readonly IFeatureManager _featureManager;

            public BetaController(IFeatureManagerSnapshot featureManager) =>
                _featureManager = featureManager;

            [FeatureGate(MyFeatureFlags.Beta)]
            public IActionResult Index() => View();
        }
    }
    ```

1. Dans *Views/_ViewImports.cshtml*, inscrivez l'assistance de balises du gestionnaire de fonctionnalités à l'aide d'une directive `@addTagHelper` :

    ```cshtml
    @addTagHelper *, Microsoft.FeatureManagement.AspNetCore
    ```

    Le code précédent permet d'utiliser l'assistance de balises `<feature>` dans les fichiers *.cshtml* du projet.

1. Ouvrez *_Layout.cshtml* dans le répertoire *Vues*\\*Partagé*. Localisez le code barres `<nav>` sous `<body>``<header>` > . Insérez une étiquette `<feature>` entre les éléments *Accueil* et *Confidentialité* de la barre de navigation, comme indiqué dans les lignes signalées ci-dessous.

    :::code language="html" source="../../includes/azure-app-configuration-navbar.md" range="15-38" highlight="13-17":::

1. Créez un répertoire *Views/Beta* et un fichier *Index.cshtml* contenant le balisage suivant :

    ```cshtml
    @{
        ViewData["Title"] = "Beta Home Page";
    }

    <h1>This is the beta website.</h1>
    ```

## <a name="build-and-run-the-app-locally"></a>Générer et exécuter l’application localement

1. Pour générer l’application à l’aide de l’interface CLI .NET Core, exécutez la commande suivante dans l’interpréteur de commandes :

    ```dotnetcli
    dotnet build
    ```

1. Une fois la génération correctement terminée, exécutez la commande suivante pour exécuter l’application web localement :

    ```dotnetcli
    dotnet run
    ```

1. Ouvrez une fenêtre de navigateur, puis accédez à `http://localhost:5000`, qui est l’URL par défaut de l’application web hébergée localement. Si vous travaillez dans Azure Cloud Shell, sélectionnez le bouton **Aperçu web**, puis **Configurer**. Quand vous y êtes invité, sélectionnez le port 5000.

    ![Rechercher le bouton Aperçu web](./media/quickstarts/cloud-shell-web-preview.png)

    Votre navigateur doit afficher une page similaire à l’image ci-dessous.

    :::image type="content" source="media/quickstarts/aspnet-core-feature-flag-local-before.png" alt-text="Application de démarrage rapide locale avant modification" border="true":::

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Toutes les ressources**, puis sélectionnez l’instance du magasin App Configuration que vous avez créée dans le guide de démarrage rapide.

1. Sélectionnez **Gestionnaire de fonctionnalités**. 

1. Activez l’indicateur *Bêta* en cochant la case située sous **Activé**.

1. Revenez à l'interface de commande. Annulez le processus `dotnet` en cours en appuyant sur <kbd>Ctrl+C</kbd>. Redémarrez votre application à l'aide de `dotnet run`.

1. Actualisez la page de navigateur pour afficher les nouveaux paramètres de configuration.

    :::image type="content" source="media/quickstarts/aspnet-core-feature-flag-local-after.png" alt-text="Application de démarrage rapide locale après modification" border="true":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE[Azure App Configuration cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un magasin App Configuration et vous l’avez utilisé pour gérer les fonctionnalités dans une application web ASP.NET Core via les [bibliothèques de gestion des fonctionnalités](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration).

* Découvrez plus d’informations sur la [gestion des fonctionnalités](./concept-feature-management.md).
* [Gérer les indicateurs de fonctionnalité](./manage-feature-flags.md).
* [Utiliser des indicateurs de fonctionnalités dans une application ASP.NET Core](./use-feature-flags-dotnet-core.md).
* [Utiliser la configuration dynamique dans une application ASP.NET Core](./enable-dynamic-configuration-aspnet-core.md)