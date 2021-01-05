---
title: Démarrage rapide pour Azure App Configuration avec ASP.NET Core | Microsoft Docs
description: Créez une application ASP.NET Core à l'aide d'Azure App Configuration pour centraliser le stockage et la gestion des paramètres d'une application ASP.NET Core.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.devlang: csharp
ms.custom: devx-track-csharp, contperf-fy21q1
ms.topic: quickstart
ms.date: 09/25/2020
ms.author: alkemper
ms.openlocfilehash: d1582cda4b083623daf7bb756d0bc8aa3e74eae8
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033644"
---
# <a name="quickstart-create-an-aspnet-core-app-with-azure-app-configuration"></a>Démarrage rapide : Créer une application ASP.NET Core avec Azure App Configuration

Dans ce guide de démarrage rapide, vous utilisez Azure App Configuration pour centraliser le stockage et la gestion des paramètres d’une application ASP.NET Core. ASP.NET Core génère un objet de configuration basé sur une paire clé-valeur unique utilisant les paramètres d’une ou de plusieurs sources de données spécifiées par une application. Ces sources de données sont appelées *fournisseurs de configuration*. Étant donné que le client .NET Core d’App Configuration est implémenté en tant que fournisseur de configuration, le service ressemble à une autre source de données.

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/dotnet)
* [Kit de développement logiciel (SDK) .NET Core](https://dotnet.microsoft.com/download)

> [!TIP]
> Azure Cloud Shell est un interpréteur de commandes interactif gratuit qui vous permet d’exécuter les instructions de ligne de commande de cet article. Les outils Azure les plus courants sont préinstallés, y compris le SDK .NET Core. Si vous êtes connecté à votre abonnement Azure, lancez [Azure Cloud Shell](https://shell.azure.com) à partir de shell.azure.com. Pour en savoir plus sur Azure Cloud Shell, consultez [notre documentation](../cloud-shell/overview.md).

## <a name="create-an-app-configuration-store"></a>Créer un magasin App Configuration

[!INCLUDE[Azure App Configuration resource creation steps](../../includes/azure-app-configuration-create.md)]

7. Sélectionnez **Opérations** > **Explorateur de configurations** > **Créer** > **Clé-valeur** pour ajouter les paires clé-valeur suivantes :

    | Clé                                | Valeur                               |
    |------------------------------------|-------------------------------------|
    | `TestApp:Settings:BackgroundColor` | *#FFF*                              |
    | `TestApp:Settings:FontColor`       | *#000*                              |
    | `TestApp:Settings:FontSize`        | *24*                                |
    | `TestApp:Settings:Message`         | *Data from Azure App Configuration* |

    Laissez **Étiquette** et **Type de contenu** vides pour l’instant. Sélectionnez **Appliquer**.

## <a name="create-an-aspnet-core-web-app"></a>Créez une application web ASP.NET Core

Utilisez l'[interface de ligne de commande (CLI) .NET Core](/dotnet/core/tools) pour créer un projet de MVC ASP.NET Core. [Azure Cloud Shell](https://shell.azure.com) vous fournit ces outils, qui sont également disponibles sur les plateformes Windows, macOS et Linux.

Exécutez la commande suivante pour créer un projet MVC ASP.NET Core dans un nouveau dossier *TestAppConfig* :

```dotnetcli
dotnet new mvc --no-https --output TestAppConfig
```

[!INCLUDE[Add Secret Manager support to an ASP.NET Core project](../../includes/azure-app-configuration-add-secret-manager.md)]

## <a name="connect-to-the-app-configuration-store"></a>Se connecter au magasin App Configuration

1. Exécutez la commande suivante pour ajouter une référence au package NuGet [Microsoft.Azure.AppConfiguration.AspNetCore](https://www.nuget.org/packages/Microsoft.Azure.AppConfiguration.AspNetCore).

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    ```

1. Exécutez la commande suivante dans le répertoire où se trouve le fichier *.csproj*. La commande utilise Secret Manager pour stocker un secret nommé `ConnectionStrings:AppConfig`, qui stocke la chaîne de connexion de votre magasin App Configuration. Remplacez l'espace réservé `<your_connection_string>` par la chaîne de connexion de votre magasin App Configuration. La chaîne de connexion se trouve sous **Clés d’accès** dans le portail Azure.

    ```dotnetcli
    dotnet user-secrets set ConnectionStrings:AppConfig "<your_connection_string>"
    ```

    > [!IMPORTANT]
    > Certains interpréteurs de commandes tronquent la chaîne de connexion, à moins qu’elle soit placée entre guillemets. Vérifiez que la sortie de la commande `dotnet user-secrets` affiche la chaîne de connexion complète. Si ce n’est pas le cas, réexécutez la commande en plaçant la chaîne de connexion entre guillemets.

    Secret Manager est utilisé uniquement pour tester l’application web localement. Une fois l'application déployée sur [Azure App Service](https://azure.microsoft.com/services/app-service/web), utilisez le paramètre d'application **Chaînes de connexion** d'App Service plutôt que Secret Manager pour stocker la chaîne de connexion.

    Accédez à ce secret à l'aide de l'API de configuration .NET Core. Avec l'API de configuration, un signe deux-points (`:`) fonctionne dans le nom de configuration sur toutes les plateformes prises en charge. Pour plus d'informations, consultez [Clés et valeurs de configuration](/aspnet/core/fundamentals/configuration#configuration-keys-and-values).

1. Dans *Program.cs*, ajoutez une référence à l’espace de noms de l’API de configuration .NET Core :

    ```csharp
    using Microsoft.Extensions.Configuration;
    ```

1. Mettez à jour la méthode `CreateWebHostBuilder` pour utiliser App Configuration en appelant la méthode `AddAzureAppConfiguration`.

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
                    config.AddAzureAppConfiguration(connection);
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
                config.AddAzureAppConfiguration(connection);
            })
            .UseStartup<Startup>();
    ```

    ---

    Avec la modification précédente, le [fournisseur de configuration d'App Configuration](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration) a été inscrit auprès de l'API de configuration .NET Core.

## <a name="read-from-the-app-configuration-store"></a>Lire depuis le magasin App Configuration

Procédez comme suit pour lire et afficher les valeurs stockées dans le stockage d’App Configuration. L’API de configuration .NET Core sera utilisée pour accéder au stockage. Syntaxe Razor sera utilisé pour afficher les valeurs des clés.

Ouvrez *\<app root>/Views/Home/Index.cshtml* et remplacez son contenu par le code suivant :

```cshtml
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<style>
    body {
        background-color: @Configuration["TestApp:Settings:BackgroundColor"]
    }
    h1 {
        color: @Configuration["TestApp:Settings:FontColor"];
        font-size: @Configuration["TestApp:Settings:FontSize"]px;
    }
</style>

<h1>@Configuration["TestApp:Settings:Message"]</h1>
```

Dans le code précédent, les clés du stockage d’App Configuration sont utilisées comme suit :

* La valeur de la clé `TestApp:Settings:BackgroundColor` est assignée à la propriété CSS `background-color`.
* La valeur de la clé `TestApp:Settings:FontColor` est assignée à la propriété CSS `color`.
* La valeur de la clé `TestApp:Settings:FontSize` est assignée à la propriété CSS `font-size`.
* La valeur de la clé `TestApp:Settings:Message` s’affiche sous la forme d’un en-tête.

## <a name="build-and-run-the-app-locally"></a>Générer et exécuter l’application localement

1. Pour générer l’application à l’aide du CLI .NET Core, accédez au répertoire racine de votre projet. Exécutez la commande suivante dans le shell de commandes :

    ```dotnetcli
    dotnet build
    ```

1. Une fois la génération terminée, exécutez la commande suivante pour exécuter l’application web localement :

    ```dotnetcli
    dotnet run
    ```

1. Si vous travaillez sur votre machine locale, utilisez un navigateur pour accéder à `http://localhost:5000`. Cette adresse est l’URL par défaut de l’application web hébergée localement. Si vous travaillez dans Azure Cloud Shell, sélectionnez le bouton **Aperçu web**, puis **Configurer**.

    ![Rechercher le bouton Aperçu web](./media/quickstarts/cloud-shell-web-preview.png)

    Lorsque vous êtes invité à configurer le port pour l’aperçu, entrez *5000*, puis sélectionnez **Ouvrir et parcourir**. La page web affichera « Données d’Azure App Configuration ».

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE[Azure App Configuration cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous :

* Avez configuré un nouveau stockage App Configuration.
* Avez enregistré un fournisseur de configuration .NET Core du stockage App Configuration.
* Avez lu les clés du stockage App Configuration avec le fournisseur de configuration.
* Avez affiché les valeurs des clés du stockage App Configuration utilisant la syntaxe Razor.

Pour savoir comment configurer votre application ASP.NET Core afin d’actualiser dynamiquement les paramètres de configuration, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Activer la configuration dynamique](./enable-dynamic-configuration-aspnet-core.md)