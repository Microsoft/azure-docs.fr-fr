---
title: 'Tutoriel : ASP.NET Core avec Azure SQL Database'
description: Découvrez comment faire fonctionner une application .NET Core dans Azure App Service, en établissant une connexion à une instance Azure SQL Database.
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 06/20/2020
ms.custom: devx-track-csharp, mvc, cli-validate, seodec18, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: e953c3f442d01c023df04c1a8af3c5fe56ea59ed
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347096"
---
# <a name="tutorial-build-an-aspnet-core-and-azure-sql-database-app-in-azure-app-service"></a>Tutoriel : Générer une application ASP.NET Core et Azure SQL Database dans Azure App Service

::: zone pivot="platform-windows"  

[Azure App Service](overview.md) offre un service d'hébergement web hautement évolutif appliquant des mises à jour correctives automatiques dans Azure. Ce tutoriel montre comment créer une application .NET Core et comment la connecter à SQL Database. Lorsque vous aurez terminé, vous disposerez d'une application MVC .NET Core fonctionnant dans App Service sous Windows.

::: zone-end

::: zone pivot="platform-linux"

[Azure App Service](overview.md) fournit un service d'hébergement web hautement évolutif et appliquant des mises à jour correctives automatiques en utilisant le système d'exploitation Linux. Ce tutoriel montre comment créer une application .NET Core et comment la connecter à une instance SQL Database. Quand vous avez terminé, vous disposez d’une application MVC .NET Core en cours d’exécution dans App Service sur Linux.

::: zone-end

![Application exécutée dans App Service](./media/tutorial-dotnetcore-sqldb-app/azure-app-in-browser.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une base de données SQL dans Azure
> * Connecter une application .NET Core à une instance SQL Database
> * Déploiement de l’application dans Azure
> * Mise à jour du modèle de données et redéploiement de l’application
> * Diffusion des journaux de diagnostic à partir d’Azure
> * Gérer l’application dans le portail Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

- <a href="https://git-scm.com/" target="_blank">Installez Git</a>
- <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Installer la dernière version du kit SDK .NET Core 3.1</a>

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-local-net-core-app"></a>Création de l’application .NET Core locale

Cette étape consiste à configurer le projet .NET Core local.

### <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Dans la fenêtre de terminal, `cd` vers un répertoire de travail.

Exécutez les commandes suivantes pour cloner l’exemple de référentiel et modifier son répertoire racine.

```bash
git clone https://github.com/azure-samples/dotnetcore-sqldb-tutorial
cd dotnetcore-sqldb-tutorial
```

Cet exemple de projet contient une simple application CRUD (Create-Read-Update-Delete) basée sur [Entity Framework Core](/ef/core/).

### <a name="run-the-application"></a>Exécution de l'application

Exécutez la commande suivante pour installer les packages requis, migrer les bases de données et démarrer l’application.

```bash
dotnet tool install -g dotnet-ef
dotnet ef database update
dotnet run
```

Dans un navigateur, accédez à `http://localhost:5000`. Sélectionnez le lien **Create New** et créez quelques éléments _to-do_.

![se connecte correctement à SQL Database](./media/tutorial-dotnetcore-sqldb-app/local-app-in-browser.png)

Pour arrêter .NET Core à tout moment, appuyez sur `Ctrl+C` dans le terminal.

## <a name="create-production-sql-database"></a>Création d’une instance SQL Database de production

Dans cette étape, vous allez créer une instance SQL Database dans Azure. Lorsque votre application est déployée sur Azure, elle utilise cette base de données cloud.

Pour l’instance SQL Database, ce tutoriel utilise [Azure SQL Database](/azure/sql-database/).

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)]

### <a name="create-a-sql-database-logical-server"></a>Créer un serveur logique SQL Database

Dans Cloud Shell, créez un serveur logique Azure SQL Database avec la commande [`az sql server create`](/cli/azure/sql/server#az-sql-server-create).

Remplacez l’espace réservé *\<server-name>* par un nom SQL Database *unique*. Ce nom est utilisé dans le point de terminaison global unique de la base de données SQL, `<server-name>.database.windows.net`. Les caractères valides sont `a`-`z`, `0`-`9`, `-`. Remplacez également *\<db-username>* et *\<db-password>* par le nom d’utilisateur et le mot de passe de votre choix. 


```azurecli-interactive
az sql server create --name <server-name> --resource-group myResourceGroup --location "West Europe" --admin-user <db-username> --admin-password <db-password>
```

Lorsque le serveur logique SQL Database est créé, l’interface Azure CLI affiche des informations similaires à l’exemple suivant :

<pre>
{
  "administratorLogin": "&lt;db-username&gt;",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "&lt;server-name&gt;.database.windows.net",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/&lt;server-name&gt;",
  "identity": null,
  "kind": "v12.0",
  "location": "westeurope",
  "name": "&lt;server-name&gt;",
  "resourceGroup": "myResourceGroup",
  "state": "Ready",
  "tags": null,
  "type": "Microsoft.Sql/servers",
  "version": "12.0"
}
</pre>

### <a name="configure-a-server-firewall-rule"></a>Configurer une règle de pare-feu du serveur

Créez une [règle de pare-feu au niveau du serveur Azure SQL Database](../azure-sql/database/firewall-configure.md) avec la commande [`az sql server firewall create`](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-create). Lorsque les adresses IP de début et de fin sont définies sur 0.0.0.0, le pare-feu est ouvert uniquement pour les autres ressources Azure. 

```azurecli-interactive
az sql server firewall-rule create --resource-group myResourceGroup --server <server-name> --name AllowAzureIps --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP] 
> Vous pouvez être encore plus restrictif dans votre règle de pare-feu en [choisissant uniquement les adresses IP sortantes que votre application utilise](overview-inbound-outbound-ips.md#find-outbound-ips).
>

Dans Cloud Shell, réexécutez la commande pour autoriser l’accès à partir de votre ordinateur local en remplaçant *\<your-ip-address>* par [votre adresse IP IPv4 locale](https://www.whatsmyip.org/).

```azurecli-interactive
az sql server firewall-rule create --name AllowLocalClient --server <server-name> --resource-group myResourceGroup --start-ip-address=<your-ip-address> --end-ip-address=<your-ip-address>
```

### <a name="create-a-database"></a>Création d'une base de données

Créez une base de données avec un [niveau de performance S0](../azure-sql/database/service-tiers-dtu.md) sur le serveur avec la commande [`az sql db create`](/cli/azure/sql/db#az-sql-db-create).

```azurecli-interactive
az sql db create --resource-group myResourceGroup --server <server-name> --name coreDB --service-objective S0
```

### <a name="create-connection-string"></a>Créer une chaîne de connexion

Obtenez la chaîne de connexion au moyen de la commande [`az sql db show-connection-string`](/cli/azure/sql/db#az-sql-db-show-connection-string).

```azurecli-interactive
az sql db show-connection-string --client ado.net --server <server-name> --name coreDB
```

Dans la sortie de la commande, remplacez *\<username>* et *\<password>* par les informations d’identification de l’administrateur de base de données que vous avez utilisées précédemment.

Chaîne de connexion pour votre application .NET Core. Copiez-la pour une utilisation ultérieure.

### <a name="configure-app-to-connect-to-production-database"></a>Configurer l’application pour qu’elle se connecte à la base de données de production

Dans votre référentiel local, ouvrez Startup.cs et recherchez le code suivant :

```csharp
services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
```

Remplacez-le par le code suivant :

```csharp
services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
```

> [!IMPORTANT]
> Pour les applications de production qui doivent effectuer un scale-out, suivez les bonnes pratiques dans [Application des migrations en production](/aspnet/core/data/ef-rp/migrations#applying-migrations-in-production).
> 

### <a name="run-database-migrations-to-the-production-database"></a>Exécuter des migrations de base de données vers la base de données de production

Votre application se connecte actuellement à une base de données SQLite locale. Maintenant que vous avez configuré une base de données SQL Azure, recréez la migration initiale pour la cibler. 

À partir de la racine du dépôt, exécutez les commandes suivantes. Remplacez *\<connection-string>* par la chaîne de connexion que vous avez créée.

```
# Delete old migrations
rm Migrations -r
# Recreate migrations
dotnet ef migrations add InitialCreate

# Set connection string to production database
# PowerShell
$env:ConnectionStrings:MyDbConnection="<connection-string>"
# CMD (no quotes)
set ConnectionStrings:MyDbConnection=<connection-string>
# Bash (no quotes)
export ConnectionStrings__MyDbConnection=<connection-string>

# Run migrations
dotnet ef database update
```

### <a name="run-app-with-new-configuration"></a>Exécuter l’application avec la nouvelle configuration

À présent que les migrations de base de données sont exécutées sur la base de données de production, testez votre application en exécutant :

```
dotnet run
```

Dans un navigateur, accédez à `http://localhost:5000`. Sélectionnez le lien **Create New** et créez quelques éléments _to-do_. Votre application, désormais, lit et écrit des données dans la base de données de production.

Validez vos modifications locales, puis validez-les dans le dépôt Git. 

```bash
git add .
git commit -m "connect to SQLDB in Azure"
```

Vous êtes maintenant prêt à déployer votre code.

## <a name="deploy-app-to-azure"></a>Déployer des applications dans Azure

Dans cette étape, vous déployez votre application .NET Core connectée à SQL Database sur App Service.

### <a name="configure-local-git-deployment"></a>Configurer le déploiement Git local

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>Créer un plan App Service

::: zone pivot="platform-windows"  

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]

::: zone-end

### <a name="create-a-web-app"></a>Créer une application web

::: zone pivot="platform-windows"  

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-dotnetcore-win-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-dotnetcore-linux-no-h.md)]

::: zone-end

### <a name="configure-connection-string"></a>Configurer la chaîne de connexion

Pour définir les chaînes de connexion de votre application Azure, utilisez la commande [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) dans Cloud Shell. Dans la commande suivante, remplacez *\<app-name>* et le paramètre *\<connection-string>* par la chaîne de connexion que vous avez créée.

```azurecli-interactive
az webapp config connection-string set --resource-group myResourceGroup --name <app-name> --settings MyDbConnection="<connection-string>" --connection-string-type SQLAzure
```

Dans ASP.NET Core, vous pouvez utiliser cette chaîne de connexion nommée (`MyDbConnection`) à l’aide du modèle standard, comme n’importe quelle chaîne de connexion spécifiée dans *appsettings. json*. Dans ce cas, `MyDbConnection` est également défini dans votre fichier *appsettings.json*. Lors de l’exécution dans App Service, la chaîne de connexion définie dans App Service est prioritaire sur celle qui est définie dans votre fichier *appsettings.json*. Le code utilise la valeur  *appsettings.json* pendant le développement local, et le même code utilise la valeur App Service lorsqu’elle est déployée.

Pour voir comment la chaîne de connexion est référencée dans votre code, consultez [Configurer l’application pour qu’elle se connecte à la base de données de production](#configure-app-to-connect-to-production-database).

### <a name="push-to-azure-from-git"></a>Effectuer une transmission de type push vers Azure à partir de Git

::: zone pivot="platform-windows"  

[!INCLUDE [push-to-azure-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Enumerating objects: 268, done.
Counting objects: 100% (268/268), done.
Compressing objects: 100% (171/171), done.
Writing objects: 100% (268/268), 1.18 MiB | 1.55 MiB/s, done.
Total 268 (delta 95), reused 251 (delta 87), pack-reused 0
remote: Resolving deltas: 100% (95/95), done.
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '64821c3558'.
remote: Generating deployment script.
remote: Project file path: .\DotNetCoreSqlDb.csproj
remote: Generating deployment script for ASP.NET MSBuild16 App
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling ASP.NET Core Web Application deployment with MSBuild16.
remote: .
remote: .
remote: .
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Triggering recycle (preview mode disabled).
remote: App container will begin restart within 10 seconds.
To https://&lt;app-name&gt;.scm.azurewebsites.net/&lt;app-name&gt;.git
 * [new branch]      master -> master
</pre>

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [push-to-azure-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Enumerating objects: 273, done.
Counting objects: 100% (273/273), done.
Delta compression using up to 4 threads
Compressing objects: 100% (175/175), done.
Writing objects: 100% (273/273), 1.19 MiB | 1.85 MiB/s, done.
Total 273 (delta 96), reused 259 (delta 88)
remote: Resolving deltas: 100% (96/96), done.
remote: Deploy Async
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'cccecf86c5'.
remote: Repository path is /home/site/repository
remote: Running oryx build...
remote: Build orchestrated by Microsoft Oryx, https://github.com/Microsoft/Oryx
remote: You can report issues at https://github.com/Microsoft/Oryx/issues
remote: .
remote: .
remote: .
remote: Done.
remote: Running post deployment command(s)...
remote: Triggering recycle (preview mode disabled).
remote: Deployment successful.
remote: Deployment Logs : 'https://&lt;app-name&gt;.scm.azurewebsites.net/newui/jsonviewer?view_url=/api/deployments/cccecf86c56493ffa594e76ea1deb3abb3702d89/log'
To https://&lt;app-name&gt;.scm.azurewebsites.net/&lt;app-name&gt;.git
 * [new branch]      master -> master
</pre>

::: zone-end

### <a name="browse-to-the-azure-app"></a>Accéder à l’application Azure

Accédez à l’application déployée à l’aide de votre navigateur web.

```bash
http://<app-name>.azurewebsites.net
```

Ajoutez quelques tâches.

![Application exécutée dans App Service](./media/tutorial-dotnetcore-sqldb-app/azure-app-in-browser.png)

**Félicitations !** Vous exécutez une application .NET Core pilotée par les données dans App Service.

## <a name="update-locally-and-redeploy"></a>Mise à jour locale et redéploiement

Dans cette étape, vous apportez une modification à votre schéma de base de données et la publiez sur Azure.

### <a name="update-your-data-model"></a>Mettre à jour votre modèle de données

Ouvrez _Models/Todo.cs_ dans l’éditeur de code. Ajoutez la propriété suivante à la classe `ToDo` :

```csharp
public bool Done { get; set; }
```

### <a name="rerun-database-migrations"></a>Réexécuter les migrations de base de données

Exécutez quelques commandes pour mettre à jour la base de données de production.

```bash
dotnet ef migrations add AddProperty
dotnet ef database update
```

> [!NOTE]
> Si vous ouvrez une nouvelle fenêtre de terminal, vous devez définir la chaîne de connexion à la base de données de production dans le terminal, comme vous l’avez fait dans [Exécuter des migrations de base de données vers la base de données de production](#run-database-migrations-to-the-production-database).
>

### <a name="use-the-new-property"></a>Utiliser la nouvelle propriété

Apportez quelques modifications à votre code pour utiliser la propriété `Done`. Pour plus de simplicité dans ce didacticiel, vous allez uniquement modifier les vues `Index` et `Create` pour voir la propriété en action.

Ouvrez _Controllers/TodosController.cs_.

Recherchez la méthode `Create([Bind("ID,Description,CreatedDate")] Todo todo)` et ajoutez `Done` à la liste des propriétés dans l’attribut `Bind`. Lorsque vous avez terminé, la signature de votre méthode `Create()` doit ressembler à ceci :

```csharp
public async Task<IActionResult> Create([Bind("ID,Description,CreatedDate,Done")] Todo todo)
```

Ouvrez _Views/Todos/Create.cshtml_.

Dans le code Razor, vous devriez voir un élément `<div class="form-group">` pour `Description` et un autre élément `<div class="form-group">` pour `CreatedDate`. Juste après ces deux éléments, ajoutez un autre élément `<div class="form-group">` pour `Done` :

```csharp
<div class="form-group">
    <label asp-for="Done" class="col-md-2 control-label"></label>
    <div class="col-md-10">
        <input asp-for="Done" class="form-control" />
        <span asp-validation-for="Done" class="text-danger"></span>
    </div>
</div>
```

Ouvrez _Views/Todos/Index.cshtml_.

Recherchez l’élément `<th></th>` vide. Juste au-dessus de cet élément, ajoutez le code Razor suivant :

```csharp
<th>
    @Html.DisplayNameFor(model => model.Done)
</th>
```

Recherchez l’élément `<td>` contenant les assistances de balise `asp-action`. Juste au-dessus de cet élément, ajoutez le code Razor suivant :

```csharp
<td>
    @Html.DisplayFor(modelItem => item.Done)
</td>
```

Voilà tout ce que vous avez à faire pour voir les modifications dans les vues `Index` et `Create`.

### <a name="test-your-changes-locally"></a>Tester vos modifications en local

Exécutez l’application localement.

```bash
dotnet run
```

> [!NOTE]
> Si vous ouvrez une nouvelle fenêtre de terminal, vous devez définir la chaîne de connexion à la base de données de production dans le terminal, comme vous l’avez fait dans [Exécuter des migrations de base de données vers la base de données de production](#run-database-migrations-to-the-production-database).
>

Dans votre navigateur, accédez à `http://localhost:5000/`. Vous pouvez maintenant ajouter un élément de tâche et cocher **Terminé**. Cette tâche devrait ensuite apparaître dans votre page d’accueil comme un élément terminé. N’oubliez pas que la vue `Edit` n’affiche pas le champ `Done`, car vous n’avez pas modifié la vue `Edit`.

### <a name="publish-changes-to-azure"></a>Publier les modifications dans Azure

```bash
git add .
git commit -m "added done field"
git push azure master
```

Une fois le `git push` terminé, accédez à votre application App Service et essayez d’ajouter un élément de tâche, puis de cocher la case **Terminé**.

![Application Azure après l’activation des Migrations Code First](./media/tutorial-dotnetcore-sqldb-app/this-one-is-done.png)

Toutes les tâches existantes sont toujours affichées. Lorsque vous republiez votre application ASP.NET Core, les données existantes dans votre base de données SQL ne sont pas perdues. En outre, Entity Framework Core Migrations modifie uniquement le schéma de données, sans toucher à vos données existantes.

## <a name="stream-diagnostic-logs"></a>Diffuser les journaux de diagnostic

Pendant l’exécution de l’application ASP.NET Core dans Azure App Service, vous pouvez acheminer les journaux d’activité de la console vers Cloud Shell. De cette façon, vous pouvez obtenir les mêmes messages de diagnostic pour vous aider à déboguer les erreurs d’application.

L’exemple de projet suit déjà les instructions fournies dans [Journalisation ASP.NET Core dans Azure](/aspnet/core/fundamentals/logging#azure-app-service-provider), et apporte deux changements à la configuration :

- Il inclut une référence à `Microsoft.Extensions.Logging.AzureAppServices` dans *DotNetCoreSqlDb.csproj*.
- Appelle `loggerFactory.AddAzureWebAppDiagnostics()` dans *Program.cs*.

Pour définir le [niveau de journalisation](/aspnet/core/fundamentals/logging#log-level) ASP.NET Core dans App Service sur `Information` à partir du niveau par défaut `Error`, utilisez la commande [`az webapp log config`](/cli/azure/webapp/log#az-webapp-log-config) dans Cloud Shell.

```azurecli-interactive
az webapp log config --name <app-name> --resource-group myResourceGroup --application-logging filesystem --level information
```

> [!NOTE]
> Le niveau de journalisation du projet est déjà défini sur `Information` dans *appsettings.json*.

Pour démarrer la diffusion de journaux, utilisez la commande [`az webapp log tail`](/cli/azure/webapp/log#az-webapp-log-tail) dans Cloud Shell.

```azurecli-interactive
az webapp log tail --name <app-name> --resource-group myResourceGroup
```

Une fois que la diffusion a démarré, actualisez l’application Azure dans le navigateur pour générer un trafic web. Vous pouvez maintenant voir les journaux d’activité de la console acheminés vers le terminal. Si vous ne voyez pas les journaux d’activité de la console, attendez 30 secondes et vérifiez à nouveau.

Pour arrêter le streaming des journaux à tout moment, appuyez sur `Ctrl`+`C`.

Pour plus d’informations sur la personnalisation des journaux d’activité ASP.NET Core, consultez [Journalisation dans ASP.NET Core](/aspnet/core/fundamentals/logging).

## <a name="manage-your-azure-app"></a>Gérer votre application Azure

Pour voir l’application que vous avez créée, dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **App Services**.

![Sélectionnez App Services dans le portail Azure](./media/tutorial-dotnetcore-sqldb-app/app-services.png)

Dans la page **App Services**, sélectionnez le nom de votre application Azure.

![Navigation au sein du portail pour accéder à l’application Azure](./media/tutorial-dotnetcore-sqldb-app/access-portal.png)

Par défaut, le portail affiche la page **Vue d’ensemble** de votre application. Cette page propose un aperçu de votre application. Ici, vous pouvez également effectuer des tâches de gestion de base (parcourir, arrêter, démarrer, redémarrer et supprimer des éléments, par exemple). Les onglets sur le côté gauche de la page affichent les différentes pages de configuration que vous pouvez ouvrir.

![Page App Service du Portail Azure](./media/tutorial-dotnetcore-sqldb-app/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une base de données SQL dans Azure
> * Connecter une application .NET Core à une instance SQL Database
> * Déploiement de l’application dans Azure
> * Mise à jour du modèle de données et redéploiement de l’application
> * Diffuser des journaux d’activité à partir d’Azure vers votre terminal
> * Gérer l’application dans le portail Azure

Passez au didacticiel suivant pour découvrir comment mapper un nom DNS personnalisé à votre application.

> [!div class="nextstepaction"]
> [Tutoriel : Mappage d’un nom DNS personnalisé à une application](app-service-web-tutorial-custom-domain.md)

Ou consultez les autres ressources :

> [!div class="nextstepaction"]
> [Configurer une application ASP.NET Core](configure-language-dotnetcore.md)
