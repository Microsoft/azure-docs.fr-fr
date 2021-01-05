---
title: Profiler des applications web ASP.NET Core Azure Linux avec Application Insights Profiler | Microsoft Docs
description: Une vue d’ensemble conceptuelle et un didacticiel pas à pas sur l’utilisation d’Application Insights Profiler.
ms.topic: conceptual
ms.custom: devx-track-csharp
author: cweining
ms.author: cweining
ms.date: 02/23/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 6ef52e946edb5db8074a9b4e3ce5e4a81ae0bde5
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561050"
---
# <a name="profile-aspnet-core-azure-linux-web-apps-with-application-insights-profiler"></a>Profiler des applications web ASP.NET Core Azure Linux avec Application Insights Profiler

Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire.

Découvrez combien de temps vous consacrez à chaque méthode de votre application web en production quand vous utilisez [Application Insights](./app-insights-overview.md). Application Insights Profiler est désormais disponible pour les applications web ASP.NET Core qui sont hébergées dans Linux sur Azure App Service. Ce guide fournit des instructions détaillées sur la façon dont les traces du profileur peuvent être collectées pour les applications web ASP.NET Core Linux.

Après avoir terminé cette procédure pas à pas, votre application peut collecter des traces du profileur telles que les traces qui sont présentées dans l’image. Dans cet exemple, la trace du profileur indique qu’une requête web particulière est lente, car le temps est passé en attente. Le *chemin réactif* dans le code qui ralentit l’application est indiqué par une icône en forme de flamme. La méthode **About** dans la section **HomeController** ralentit l’application web, car la méthode appelle la fonction **Thread.Sleep**.

![Traces du profileur](./media/profiler-aspnetcore-linux/profiler-traces.png)

## <a name="prerequisites"></a>Prérequis
Les instructions suivantes s’appliquent à tous les environnements de développement Windows, Linux et Mac :

* Installez le [kit SDK .NET Core 2.1.2 ou une version ultérieure](https://dotnet.microsoft.com/download/archives).
* Installez Git en suivant les instructions dans [Prise en main - Installation de Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

## <a name="set-up-the-project-locally"></a>Configurer le projet localement

1. Ouvrez une fenêtre d’invite de commandes sur votre machine. Les instructions suivantes sont valables pour tous les environnements de développement Windows, Linux et Mac.

1. Créez une application web ASP.NET Core MVC :

   ```console
   dotnet new mvc -n LinuxProfilerTest
   ```

1. Modifiez le répertoire de travail sur le dossier racine pour le projet.

1. Ajoutez le package NuGet pour collecter les traces du profileur :

   ```console
   dotnet add package Microsoft.ApplicationInsights.Profiler.AspNetCore
   ```

1. Activez Application Insights dans Program.cs :

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseApplicationInsights() // Add this line of code to Enable Application Insights
            .UseStartup<Startup>();
    ```

1. Activer Profiler dans Startup.cs :

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddServiceProfiler(); // Add this line of code to Enable Profiler
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
    }
    ```

1. Ajoutez une ligne de code dans la section **HomeController.cs** pour retarder de quelques secondes de façon aléatoire :

    ```csharp
    using System.Threading;
    ...

    public IActionResult About()
        {
            Random r = new Random();
            int delay = r.Next(5000, 10000);
            Thread.Sleep(delay);
            return View();
        }
    ```

1. Enregistrez et validez vos modifications dans le référentiel local :

    ```console
    git init
    git add .
    git commit -m "first commit"
    ```

## <a name="create-the-linux-web-app-to-host-your-project"></a>Créer l’application web Linux pour héberger votre projet

1. Créez l’environnement d’application web à l’aide d’App Service sur Linux :

    ![Créer l’application web Linux](./media/profiler-aspnetcore-linux/create-linux-appservice.png)

2. Créez les informations d’identification de déploiement :

    > [!NOTE]
    > Enregistrez votre mot de passe pour une utilisation ultérieure lors du déploiement de votre application web.

    ![Créer les informations d’identification de déploiement](./media/profiler-aspnetcore-linux/create-deployment-credentials.png)

3. Choisissez les options de déploiement. Configurez un référentiel Git local dans l’application web en suivant les instructions sur le portail Azure. Un référentiel Git est créé automatiquement.

    ![Configurer le référentiel Git](./media/profiler-aspnetcore-linux/setup-git-repo.png)

Pour plus d’options de déploiement, consultez la [documentation App Service](../../app-service/index.yml).

## <a name="deploy-your-project"></a>Déployez votre projet

1. Dans la fenêtre d’invite de commandes, accédez au dossier racine de votre projet. Ajoutez un référentiel distant Git de façon à pointer vers le référentiel d’App Service :

    ```console
    git remote add azure https://<username>@<app_name>.scm.azurewebsites.net:443/<app_name>.git
    ```

    * Utilisez le **nom d’utilisateur** que vous avez utilisé pour créer les informations d’identification de déploiement.
    * Utilisez le **nom de l’application** que vous avez utilisé pour créer l’application web à l’aide d’App Service sur Linux.

2. Déployez le projet en envoyant les modifications vers Azure :

    ```console
    git push azure main
    ```

    Vous devez obtenir une sortie similaire à la suivante :

    ```output
    Counting objects: 9, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (8/8), done.
    Writing objects: 100% (9/9), 1.78 KiB | 911.00 KiB/s, done.
    Total 9 (delta 3), reused 0 (delta 0)
    remote: Updating branch 'main'.
    remote: Updating submodules.
    remote: Preparing deployment for commit id 'd7369a99d7'.
    remote: Generating deployment script.
    remote: Running deployment command...
    remote: Handling ASP.NET Core Web Application deployment.
    remote: ......
    remote:   Restoring packages for /home/site/repository/EventPipeExampleLinux.csproj...
    remote: .
    remote:   Installing Newtonsoft.Json 10.0.3.
    remote:   Installing Microsoft.ApplicationInsights.Profiler.Core 1.1.0-LKG
    ...
    ```

## <a name="add-application-insights-to-monitor-your-web-apps"></a>Ajouter Application Insights pour contrôler vos applications web

1. [Créez une ressource Application Insights](./create-new-resource.md).

2. Copiez la valeur **iKey** de la ressource Application Insights et définissez les paramètres suivants dans vos applications web :

    `APPINSIGHTS_INSTRUMENTATIONKEY: [YOUR_APPINSIGHTS_KEY]`

    Lorsque les paramètres de l’application sont modifiés, le site redémarre automatiquement. Une fois que les nouveaux paramètres sont appliqués, le profileur s’exécute immédiatement pendant deux minutes. Le profileur s’exécute ensuite toutes les heures pendant deux minutes.

3. Générer du trafic vers votre site web. Vous pouvez générer du trafic en actualisant plusieurs fois la page **About** du site.

4. Patientez 2 à 5 minutes pendant l’agrégation des événements dans Application Insights.

5. Accédez au volet **Niveau de performance** d’Application Insights dans le portail Azure. Vous pouvez voir les traces du profileur en bas à droite du volet.

    ![Afficher les traces du profileur](./media/profiler-aspnetcore-linux/view-traces.png)



## <a name="next-steps"></a>Étapes suivantes
Si vous utilisez des conteneurs personnalisés qui sont hébergés par Azure App Service, suivez les instructions dans [Activer le profileur de service pour l’application ASP.NET Core en conteneur](https://github.com/Microsoft/ApplicationInsights-Profiler-AspNetCore/tree/master/examples/EnableServiceProfilerForContainerApp) pour activer Application Insights Profiler.

Signalez tout problème ou suggestion sur le dépôt GitHub Application Insights : [ApplicationInsights-Profiler-AspNetCore: Issues](https://github.com/Microsoft/ApplicationInsights-Profiler-AspNetCore/issues).