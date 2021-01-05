---
title: Supprimer Application Insights dans Visual Studio - Azure Monitor
description: Guide pratique pour supprimer le kit SDK Application Insights pour ASP.NET et ASP.NET Core dans Visual Studio.
ms.topic: conceptual
ms.date: 04/06/2020
ms.openlocfilehash: 5bfa6ee21cc1a55f653c0e79807a14ac34082e73
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90981479"
---
# <a name="how-to-remove-application-insights-in-visual-studio"></a>Guide pratique pour supprimer Application Insights dans Visual Studio

Cet article vous montre comment supprimer le kit SDK Application Insights pour ASP.NET et ASP.NET Core dans Visual Studio.

Pour supprimer Application Insights, vous devez supprimer les packages NuGet et les références de l’API dans votre application. Vous pouvez désinstaller les packages NuGet à l’aide de la console de gestion des packages ou de Gérer les packages NuGet pour la solution dans Visual Studio. Les sections suivantes présentent deux façons de supprimer les packages NuGet et les éléments ajoutés automatiquement à votre projet. Veillez à confirmer que les fichiers ajoutés et les zones dans votre propre code où vous avez effectué des appels à l’API sont supprimés.

## <a name="uninstall-using-the-package-management-console"></a>Désinstaller à l’aide de la console de gestion des packages

# <a name="net"></a>[.NET](#tab/net)

1. Pour ouvrir la console de gestion des packages, sélectionnez dans le menu supérieur Outils > Gestionnaire de package NuGet > Console du Gestionnaire de package.
     
    ![Dans le menu supérieur, cliquez sur Outils > Gestionnaire de package NuGet > Console du Gestionnaire de package](./media/remove-application-insights/package-manager.png)

    > [!NOTE]
    > Si la collecte des traces est activée, vous devez d’abord désinstaller Microsoft.ApplicationInsights.TraceListener. Entrez `Uninstall-package Microsoft.ApplicationInsights.TraceListener` et suivez l’étape ci-dessous pour supprimer Microsoft.ApplicationInsights.Web.

1. Entrez la commande suivante :  `Uninstall-Package Microsoft.ApplicationInsights.Web -RemoveDependencies`

    Après avoir entré la commande, le package Application Insights et toutes ses dépendances sont désinstallés du projet.
    
    ![Entrer la commande dans la console](./media/remove-application-insights/package-management-console.png)

# <a name="net-core"></a>[.NET Core](#tab/netcore)

1. Pour ouvrir la console de gestion des packages, sélectionnez dans le menu supérieur Outils > Gestionnaire de package NuGet > Console du Gestionnaire de package.

    ![Dans le menu supérieur, cliquez sur Outils > Gestionnaire de package NuGet > Console du Gestionnaire de package](./media/remove-application-insights/package-manager.png)

1. Entrez la commande suivante :  ` Uninstall-Package Microsoft.ApplicationInsights.AspNetCore -RemoveDependencies`

    Après avoir entré la commande, le package Application Insights et toutes ses dépendances sont désinstallés du projet.

---

## <a name="uninstall-using-the-visual-studio-nugetui"></a>Désinstaller à l’aide de l’interface utilisateur NuGet Visual Studio

# <a name="net"></a>[.NET](#tab/net)

1. Dans l’ *Explorateur de solutions* à droite, cliquez avec le bouton droit sur **Solution** et sélectionnez **Gérer les packages NuGet pour la solution**.

    Un écran s’affiche alors pour vous permettre de modifier tous les packages NuGet qui font partie du projet.
    
     ![Cliquez avec le bouton droit sur Solution dans l’Explorateur de solutions, puis sélectionnez Gérer les packages NuGet pour la solution](./media/remove-application-insights/manage-nuget-framework.png)

    > [!NOTE]
    > Si la collecte des traces est activée, vous devez d’abord désinstaller Microsoft.ApplicationInsights.TraceListener sans sélectionner l’option Supprimer les dépendances, puis suivre les étapes ci-dessous pour désinstaller Microsoft.ApplicationInsights.Web en sélectionnant l’option Supprimer les dépendances.
    
1. Cliquez sur le package « Microsoft.ApplicationInsights.Web ». Sur la droite, cochez la case en regard de *Projet* pour sélectionner tous les projets.
    
1. Pour supprimer toutes les dépendances lors de la désinstallation, sélectionnez le bouton de liste déroulante **Options** sous la section où vous avez sélectionné Projet.

    Sous *Options de désinstallation*, cochez la case en regard de *Supprimer les dépendances*.

1. Sélectionner **Désinstaller**.
    
    ![La capture d’écran montre la fenêtre Microsoft.ApplicationInsights.Web avec l’option Supprimer les dépendances sélectionnée et Désinstaller en surbrillance.](./media/remove-application-insights/uninstall-framework.png)

    Une boîte de dialogue s’affiche pour montrer toutes les dépendances à supprimer de l’application. Sélectionnez **OK** pour désinstaller.
    
    ![La capture d’écran montre une boîte de dialogue avec les dépendances à supprimer.](./media/remove-application-insights/preview-uninstall-framework.png)
    
1.  Une fois que tout est désinstallé, vous pouvez toujours voir « ApplicationInsights.config » et « AiHandleErrorAttribute.cs » dans l’ *Explorateur de solutions*. Vous pouvez supprimer les deux fichiers manuellement.

# <a name="net-core"></a>[.NET Core](#tab/netcore)

1. Dans l’ *Explorateur de solutions* à droite, cliquez avec le bouton droit sur **Solution** et sélectionnez **Gérer les packages NuGet pour la solution**.

    Un écran s’affiche alors pour vous permettre de modifier tous les packages NuGet qui font partie du projet.

    ![Cliquez avec le bouton droit sur Solution dans l’Explorateur de solutions, puis sélectionnez Gérer les packages NuGet pour la solution](./media/remove-application-insights/manage-nuget-core.png)

1. Cliquez sur le package « Microsoft.ApplicationInsights.AspNetCore ». Sur la droite, cochez la case en regard de *Projet* pour sélectionner tous les projets, puis choisissez **Désinstaller**.

    ![Cocher Supprimer les dépendances, puis désinstaller](./media/remove-application-insights/uninstall-core.png)

---

## <a name="what-is-created-when-you-add-application-insights"></a>Éléments créés quand vous ajoutez Application Insights

Lorsque vous ajoutez Application Insights à votre projet, il crée des fichiers et ajoute du code à certains de vos fichiers. La désinstallation seule des packages NuGet n’entraîne pas toujours celle des fichiers et du code. Pour supprimer complètement Application Insights, vous devez vérifier et supprimer manuellement le code ou les fichiers ajoutés ainsi que tous les appels d’API que vous avez ajoutés dans votre projet.

# <a name="net"></a>[.NET](#tab/net)

Lorsque vous ajoutez Application Insights Telemetry à un projet Visual Studio ASP.NET, les fichiers suivants sont ajoutés :

- ApplicationInsights.config
- AiHandleErrorAttribute.cs

Les éléments de code suivants sont ajoutés :

- [Nom de votre projet].csproj

    ```C#
     <ApplicationInsightsResourceId>/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Default-ApplicationInsights-EastUS/providers/microsoft.insights/components/WebApplication4</ApplicationInsightsResourceId>
    ```

- Packages.config

    ```xml
    <packages>
    ...
    
      <package id="Microsoft.ApplicationInsights" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.Agent.Intercept" version="2.4.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.DependencyCollector" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.PerfCounterCollector" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.Web" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.WindowsServer" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel" version="2.12.0" targetFramework="net472" />
    
      <package id="Microsoft.AspNet.TelemetryCorrelation" version="1.0.7" targetFramework="net472" />
    
      <package id="System.Buffers" version="4.4.0" targetFramework="net472" />
      <package id="System.Diagnostics.DiagnosticSource" version="4.6.0" targetFramework="net472" />
      <package id="System.Memory" version="4.5.3" targetFramework="net472" />
      <package id="System.Numerics.Vectors" version="4.4.0" targetFramework="net472" />
      <package id="System.Runtime.CompilerServices.Unsafe" version="4.5.2" targetFramework="net472" />
    ...
    </packages>
    ```

- Layout.cshtml

    Si votre projet comporte un fichier Layout.cshtml, le code ci-dessous est ajouté.
    
    ```html
    <head>
    ...
        <script type = 'text/javascript' >
            var appInsights=window.appInsights||function(config)
            {
                function r(config){ t[config] = function(){ var i = arguments; t.queue.push(function(){ t[config].apply(t, i)})} }
                var t = { config:config},u=document,e=window,o='script',s=u.createElement(o),i,f;for(s.src=config.url||'//az416426.vo.msecnd.net/scripts/a/ai.0.js',u.getElementsByTagName(o)[0].parentNode.appendChild(s),t.cookie=u.cookie,t.queue=[],i=['Event','Exception','Metric','PageView','Trace','Ajax'];i.length;)r('track'+i.pop());return r('setAuthenticatedUserContext'),r('clearAuthenticatedUserContext'),config.disableExceptionTracking||(i='onerror',r('_'+i),f=e[i],e[i]=function(config, r, u, e, o) { var s = f && f(config, r, u, e, o); return s !== !0 && t['_' + i](config, r, u, e, o),s}),t
            }({
                instrumentationKey:'00000000-0000-0000-0000-000000000000'
            });
            
            window.appInsights=appInsights;
            appInsights.trackPageView();
        </script>
    ...
    </head>
    ```

- ConnectedService.json

    ```json
    {
      "ProviderId": "Microsoft.ApplicationInsights.ConnectedService.ConnectedServiceProvider",
      "Version": "16.0.0.0",
      "GettingStartedDocument": {
        "Uri": "https://go.microsoft.com/fwlink/?LinkID=613413"
      }
    }
    ```

- FilterConfig.cs

    ```csharp
            public static void RegisterGlobalFilters(GlobalFilterCollection filters)
            {
                filters.Add(new ErrorHandler.AiHandleErrorAttribute());// This line was added
            }
    ```

# <a name="net-core"></a>[.NET Core](#tab/netcore)

Lorsque vous ajoutez Application Insights Telemetry à un projet de modèle Visual Studio ASP.NET Core, le code suivant est ajouté :

- [Nom de votre projet].csproj

    ```csharp
      <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <ApplicationInsightsResourceId>/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Default-ApplicationInsights-EastUS/providers/microsoft.insights/components/WebApplication4core</ApplicationInsightsResourceId>
      </PropertyGroup>
    
      <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.12.0" />
      </ItemGroup>
    
      <ItemGroup>
        <WCFMetadata Include="Connected Services" />
      </ItemGroup>
    ```

- Appsettings.json

    ```json
    "ApplicationInsights": {
        "InstrumentationKey": "00000000-0000-0000-0000-000000000000"
    ```

- ConnectedService.json
    
    ```json
    {
      "ProviderId": "Microsoft.ApplicationInsights.ConnectedService.ConnectedServiceProvider",
      "Version": "16.0.0.0",
      "GettingStartedDocument": {
        "Uri": "https://go.microsoft.com/fwlink/?LinkID=798432"
      }
    }
    ```
- Startup.cs

    ```csharp
       public void ConfigureServices(IServiceCollection services)
            {
                services.AddRazorPages();
                services.AddApplicationInsightsTelemetry(); // This is added
            }
    ```

---

## <a name="next-steps"></a>Étapes suivantes

- [Azure Monitor](../overview.md)
