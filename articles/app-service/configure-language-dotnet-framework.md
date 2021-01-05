---
title: Configurer des applications ASP.NET
description: Découvrez comment configurer une application ASP.NET dans Azure App Service. Cet article présente les tâches de configuration les plus courantes.
ms.devlang: dotnet
ms.custom: devx-track-csharp, devx-track-azurecli
ms.topic: article
ms.date: 06/02/2020
ms.openlocfilehash: 8ed6835583cc4881b19eee14ed392b193324535e
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92744161"
---
# <a name="configure-an-aspnet-app-for-azure-app-service"></a>Configurer une application ASP.NET pour Azure App Service

> [!NOTE]
> Pour ASP.NET Core, consultez la section [Configurer une application ASP.NET Core pour Azure App Service](configure-language-dotnetcore.md)

Les applications ASP.NET doivent être déployées pour Azure App Service en tant que fichiers binaires compilés. L’outil de publication de Visual Studio génère la solution, puis déploie directement les fichiers binaires compilés, alors que le moteur de déploiement d’App Service déploie en premier le référentiel de code, puis compile les fichiers binaires.

Ce guide fournit des concepts clés et des instructions pour les développeurs ASP.NET. Si vous n’avez jamais utilisé Azure App Service, commencez par suivre le [Guide de démarrage rapide d’ASP.NET](quickstart-dotnet-framework.md) et le [tutoriel sur ASP.NET avec SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md).

## <a name="show-supported-net-framework-runtime-versions"></a>Afficher les versions du runtime .NET Framework prises en charge

Dans App Service, toutes les versions de .NET Framework prises en charge sont déjà installées sur les instances Windows. Pour afficher le runtime .NET Framework et les versions du Kit de développement logiciel (SDK) disponibles, accédez à `https://<app-name>.scm.azurewebsites.net/DebugConsole` et exécutez la commande appropriée dans la console basée sur navigateur :

Pour les versions du runtime CLR 4 (.NET Framework 4 et versions ultérieures) :

```CMD
ls "D:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\"
```

La dernière version de .NET Framework n’est peut-être pas disponible immédiatement.

Pour les versions du runtime CLR 2 (.NET Framework 3.5 et versions antérieures) :

```CMD
ls "D:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\"
```

## <a name="show-current-net-framework-runtime-version"></a>Afficher la version du runtime .NET Framework actuelle

Exécutez la commande suivante dans le [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query netFrameworkVersion
```

Une valeur de `v4.0` signifie que la dernière version de CLR 4 (.NET Framework 4.x) est utilisée. La valeur de `v2.0` signifie qu’une version CLR 2 (.NET Framework 3.5) est utilisée.

## <a name="set-net-framework-runtime-version"></a>Définir la version du runtime .NET Framework

Par défaut, App Service utilise la dernière version de .NET Framework prise en charge pour exécuter votre application ASP.NET. Pour exécuter votre application avec .NET Framework 3.5, exécutez la commande suivante dans le [Cloud Shell](https://shell.azure.com) (v2.0 signifie CLR 2) :

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --net-framework-version v2.0
```

## <a name="access-environment-variables"></a>Accéder aux variables d’environnement

Dans App Service, vous pouvez [définir les paramètres de l’application](configure-common.md#configure-app-settings) et les chaînes de connexion en dehors de votre code d’application. Vous pouvez ensuite y accéder dans n’importe quelle classe à l’aide du modèle ASP.NET standard :

```csharp
using System.Configuration;
...
// Get an app setting
ConfigurationManager.AppSettings["MySetting"];
// Get a connection string
ConfigurationManager.ConnectionStrings["MyConnection"];
}
```

Si vous configurez un paramètre d’application portant le même nom dans App Service et dans *web.config* , la valeur d’App Service est prioritaire sur la valeur de *web.config* . La valeur de *web.config* locale vous permet de déboguer l’application localement, tandis que la valeur d’App Service vous permet d’exécuter l’application dans le produit avec les paramètres de production. Les chaînes de connexion fonctionnent de la même façon. De cette façon, vous pouvez conserver les secrets de votre application en dehors de votre référentiel de code et accéder aux valeurs appropriées sans modifier votre code.

## <a name="deploy-multi-project-solutions"></a>Déployer des solutions à projets multiples

Quand une solution Visual Studio comprend plusieurs projets, le processus de publication de Visual Studio comprend déjà la sélection du projet à déployer. Lorsque vous effectuez un déploiement vers le moteur de déploiement App Service, comme c’est le cas avec Git ou Zip Deploy, avec l’automation de la génération activée, le moteur de déploiement App Service sélectionne le premier site web ou projet d’application web qu’il trouve en tant qu’application App Service. Vous pouvez spécifier le projet qu’App Service doit utiliser en spécifiant le paramètre d’application `PROJECT`. Par exemple, exécutez la commande suivante dans le service [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp config appsettings set --resource-group <resource-group-name> --name <app-name> --settings PROJECT="<project-name>/<project-name>.csproj"
```

## <a name="get-detailed-exceptions-page"></a>Accéder à la page d’exceptions détaillées

Lorsque votre application ASP.NET génère une exception dans le débogueur Visual Studio, le navigateur affiche une page d’exceptions détaillées, mais dans App Service, cette page est remplacée par un message d’erreur générique. Pour afficher la page d’exceptions détaillées dans App Service, ouvrez le fichier *Web.config* et ajoutez l’élément `<customErrors mode="Off"/>` sous l’élément `<system.web>`. Par exemple :

```xml
<system.web>
    <customErrors mode="Off"/>
</system.web>
```

Redéployez votre application avec le fichier *Web.config* mis à jour. La même page d’exceptions détaillées est maintenant affichée.

## <a name="access-diagnostic-logs"></a>Accéder aux journaux de diagnostic

Vous pouvez ajouter des messages de diagnostic dans votre code d’application à l’aide de [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace). Par exemple : 

```csharp
Trace.TraceError("Record not found!"); // Error trace
Trace.TraceWarning("Possible data loss"); // Warning trace
Trace.TraceInformation("GET /Home/Index"); // Information trace
```

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Créer une application ASP.NET dans Azure avec SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md)
