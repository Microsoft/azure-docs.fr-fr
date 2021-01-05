---
title: 'Tutoriel : Créer une application web (Cache-Aside) - Cache Azure for Redis'
description: Découvrez comment créer une application web qui utilise le modèle Cache-Aside avec le cache Azure for Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: tutorial
ms.custom: devx-track-csharp, mvc
ms.date: 03/30/2018
ms.openlocfilehash: 90e60044e227ea1a18ea032d302b29abda1ea2e8
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536842"
---
# <a name="tutorial-create-a-cache-aside-leaderboard-on-aspnet"></a>Tutoriel : Créer un leaderboard de type Cache-Aside sur ASP.NET

Dans ce tutoriel, vous allez mettre à jour l’application web ASP.NET *ContosoTeamStats* , créée dans le [guide de démarrage rapide ASP.NET pour Cache Azure pour Redis](cache-web-app-howto.md), afin d’inclure un leaderboard qui utilise le [modèle Cache-Aside](/azure/architecture/patterns/cache-aside) avec Cache Azure pour Redis. L’exemple d’application affiche une liste des statistiques d’équipe d’une base de données et montre les différentes façons d’utiliser Cache Azure pour Redis afin de stocker et de récupérer des données à partir du cache pour améliorer les performances. Une fois le tutoriel terminé, vous disposerez d’une application web, optimisée avec Cache Azure pour Redis et hébergée dans Azure, effectuant des opérations de lecture et écriture sur une base de données.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Améliorer le débit des données et réduire la charge de la base de données en stockant et en récupérant des données à l’aide de Cache Azure pour Redis.
> * Utiliser un ensemble trié Redis pour récupérer les cinq meilleures équipes.
> * Approvisionner des ressources Azure pour l’application à l’aide d’un modèle Resource Manager.
> * Publier l’application sur Azure avec Visual Studio.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez disposer des éléments suivants :

* Ce tutoriel se poursuit là où vous l’avez laissé dans le [guide de démarrage rapide ASP.NET pour Cache Azure pour Redis](cache-web-app-howto.md). Si vous ne l’avez pas encore fait, commencez par suivre ce guide de démarrage rapide.
* Installez [Visual Studio 2019](https://www.visualstudio.com/downloads/) avec les charges de travail suivantes :
    * Développement web et ASP.NET
    * Développement Azure
    * Développement de bureau .NET avec la base de données locale SQL Server Express ou [l’édition SQL Server 2017 Express](https://www.microsoft.com/sql-server/sql-server-editions-express).

## <a name="add-a-leaderboard-to-the-project"></a>Ajouter un leaderboard au projet

Dans cette section du didacticiel, vous configurez le projet *ContosoTeamStats* avec un leaderboard qui signale les statistiques de gain, de perte et de lien pour obtenir une liste des équipes fictives.

### <a name="add-the-entity-framework-to-the-project"></a>Ajouter Entity Framework au projet

1. Dans Visual Studio, ouvrez la solution *ContosoTeamStats* que vous avez créée dans le [guide de démarrage rapide ASP.NET pour Cache Azure pour Redis](cache-web-app-howto.md).
2. Cliquez sur **Outils > Gestionnaire de package NuGet > Console du gestionnaire de package** .
3. Exécutez la commande suivante dans la fenêtre **Console du Gestionnaire de package** pour installer EntityFramework :

    ```powershell
    Install-Package EntityFramework
    ```

Pour plus d’informations sur ce package, consultez la page NuGet [EntityFramework](https://www.nuget.org/packages/EntityFramework/).

### <a name="add-the-team-model"></a>Ajouter le modèle d’équipe

1. Cliquez avec le bouton droit sur **Modèles** dans l’ **Explorateur de solutions** et sélectionnez **Ajouter** , **Classe** .

1. Entrez le nom de classe `Team` et cliquez sur **Ajouter** .

    ![Ajouter une classe de modèle](./media/cache-web-app-cache-aside-leaderboard/cache-model-add-class-dialog.png)

1. Remplacez les instructions `using` au début du fichier *Team.cs* par les instructions `using` suivantes :

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Data.Entity.SqlServer;
    ```

1. Remplacez la définition de la classe `Team` par l’extrait de code suivant, qui contient une définition de classe `Team` mise à jour, ainsi que d’autres classes d’assistance Entity Framework. Ce didacticiel utilise l’approche Code First avec Entity Framework. Cette approche permet à Entity Framework de créer la base de données à partir de votre code. Pour plus d’informations sur l’approche Code First d’Entity Framework utilisée dans ce didacticiel, consultez [Code First pour une nouvelle base de données](/ef/ef6/modeling/code-first/workflows/new-database).

    ```csharp
    public class Team
    {
        public int ID { get; set; }
        public string Name { get; set; }
        public int Wins { get; set; }
        public int Losses { get; set; }
        public int Ties { get; set; }

        static public void PlayGames(IEnumerable<Team> teams)
        {
            // Simple random generation of statistics.
            Random r = new Random();

            foreach (var t in teams)
            {
                t.Wins = r.Next(33);
                t.Losses = r.Next(33);
                t.Ties = r.Next(0, 5);
            }
        }
    }

    public class TeamContext : DbContext
    {
        public TeamContext()
            : base("TeamContext")
        {
        }

        public DbSet<Team> Teams { get; set; }
    }

    public class TeamInitializer : CreateDatabaseIfNotExists<TeamContext>
    {
        protected override void Seed(TeamContext context)
        {
            var teams = new List<Team>
            {
                new Team{Name="Adventure Works Cycles"},
                new Team{Name="Alpine Ski House"},
                new Team{Name="Blue Yonder Airlines"},
                new Team{Name="Coho Vineyard"},
                new Team{Name="Contoso, Ltd."},
                new Team{Name="Fabrikam, Inc."},
                new Team{Name="Lucerne Publishing"},
                new Team{Name="Northwind Traders"},
                new Team{Name="Consolidated Messenger"},
                new Team{Name="Fourth Coffee"},
                new Team{Name="Graphic Design Institute"},
                new Team{Name="Nod Publishers"}
            };

            Team.PlayGames(teams);

            teams.ForEach(t => context.Teams.Add(t));
            context.SaveChanges();
        }
    }

    public class TeamConfiguration : DbConfiguration
    {
        public TeamConfiguration()
        {
            SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
        }
    }
    ```

1. Dans l’ **Explorateur de solutions** , double-cliquez sur le fichier **Web.config** pour l’ouvrir.

    ![Web.config](./media/cache-web-app-cache-aside-leaderboard/cache-web-config.png)

1. Ajoutez la section `connectionStrings` suivante dans la section `configuration`. Le nom de la chaîne de connexion doit correspondre au nom de la classe de contexte de base de données Entity Framework, qui est `TeamContext`.

    Cette chaîne de connexion part du principe que vous avez rempli les [conditions préalables](#prerequisites) et installé la base de données locale SQL Server Express, qui fait partie de la charge de travail de *développement de bureau .NET* installée avec Visual Studio 2019.

    ```xml
    <connectionStrings>
        <add name="TeamContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>
    ```

    L’exemple suivant montre la nouvelle section `connectionStrings` suivant `configSections` à l’intérieur de la section `configuration` :

    ```xml
    <configuration>
        <configSections>
        <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
        </configSections>
        <connectionStrings>
        <add name="TeamContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True"     providerName="System.Data.SqlClient" />
        </connectionStrings>
        ...
    ```

### <a name="add-the-teamscontroller-and-views"></a>Ajouter les TeamsController et les vues

1. Dans Visual Studio, créez le projet. 

1. Dans l’ **Explorateur de solutions** , cliquez avec le bouton droit sur le dossier **Contrôleurs** . Cliquez ensuite sur **Ajouter** puis sur **Contrôleur** .

1. Sélectionnez **Contrôleur MVC 5 avec vues, en utilisant Entity Framework** , puis cliquez sur **Ajouter** . Si vous obtenez une erreur après avoir cliqué sur **Ajouter** , assurez-vous que vous avez déjà généré le projet.

    ![Ajouter une classe de contrôleur](./media/cache-web-app-cache-aside-leaderboard/cache-add-controller-class.png)

1. Sélectionnez **Team (ContosoTeamStats.Models)** dans la liste déroulante **Classe de modèle** . Sélectionnez **TeamContext (ContosoTeamStats.Models)** dans la liste déroulante **Classe du contexte de données** . Tapez `TeamsController` dans la zone de texte **Nom du contrôleur** (si elle n’est pas remplie automatiquement). Cliquez sur **Ajouter** pour créer la classe de contrôleur et ajouter les vues par défaut.

    ![Configurer un contrôleur](./media/cache-web-app-cache-aside-leaderboard/cache-configure-controller.png)

1. Dans l’ **Explorateur de solutions** , développez **Global.asax** , puis double-cliquez sur **Global.asax.cs** pour l’ouvrir.

    ![Global.asax.cs](./media/cache-web-app-cache-aside-leaderboard/cache-global-asax.png)

1. Ajoutez les deux instructions `using` suivantes au début du fichier, sous les autres instructions `using` :

    ```csharp
    using System.Data.Entity;
    using ContosoTeamStats.Models;
    ```

1. Ajoutez la ligne de code ci-après à la fin de la méthode `Application_Start` :

    ```csharp
    Database.SetInitializer<TeamContext>(new TeamInitializer());
    ```

1. Dans l’ **Explorateur de solutions** , développez `App_Start` et double-cliquez sur `RouteConfig.cs`.

    ![RouteConfig.cs](./media/cache-web-app-cache-aside-leaderboard/cache-RouteConfig-cs.png)

1. Dans la méthode `controller = "Home"`, remplacez `RegisterRoutes` par la route `Default` avec `controller = "Teams"` comme indiqué dans l’exemple suivant :

    ```csharp
    routes.MapRoute(
        name: "Default",
        url: "{controller}/{action}/{id}",
        defaults: new { controller = "Teams", action = "Index", id = UrlParameter.Optional }
    );
    ```

### <a name="configure-the-layout-view"></a>Configurer l’affichage de la mise en page

1. Dans l’ **Explorateur de solutions** , développez le dossier **Vues** puis le dossier **Partagé** et double-cliquez sur **_Layout.cshtml** . 

    ![_Layout.cshtml](./media/cache-web-app-cache-aside-leaderboard/cache-layout-cshtml.png)

1. Modifiez le contenu de l’élément `title` et remplacez `My ASP.NET Application` par `Contoso Team Stats`, comme indiqué dans l’exemple suivant :

    ```html
    <title>@ViewBag.Title - Contoso Team Stats</title>
    ```

1. Dans la section `body`, ajoutez la nouvelle instruction `Html.ActionLink` suivante pour les *statistiques d’équipe Contoso* juste en dessous du lien pour le *test Cache Azure pour Redis* .

    ```csharp
    @Html.ActionLink("Contoso Team Stats", "Index", "Teams", new { area = "" }, new { @class = "navbar-brand" })`
    ```

    ![Modifications du code](./media/cache-web-app-cache-aside-leaderboard/cache-layout-cshtml-code.png)

1. Appuyez sur **Ctrl+F5** pour générer et exécuter l’application. Cette version de l’application lit les résultats directement à partir de la base de données. Notez les actions **Créer** , **Modifier** , **Détails** et **Supprimer** qui ont été automatiquement ajoutées à l’application par le modèle automatique **Contrôleur MVC 5 avec vues, en utilisant Entity Framework** . Dans la section suivante du tutoriel, vous allez ajouter Cache Azure pour Redis afin d’optimiser l’accès aux données et de fournir des fonctionnalités supplémentaires à l’application.

    ![Application de départ](./media/cache-web-app-cache-aside-leaderboard/cache-starter-application.png)

## <a name="configure-the-app-for-azure-cache-for-redis"></a>Configurer l’application pour Cache Azure pour Redis

Dans cette section du tutoriel, vous allez configurer l’exemple d’application pour stocker et récupérer des statistiques d’équipe Contoso à partir d’une instance de Cache Azure pour Redis à l’aide du client de cache [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis).

### <a name="add-a-cache-connection-to-the-teams-controller"></a>Ajoutez une connexion de cache au contrôleur Teams

Vous avez déjà installé le package de bibliothèque de client *StackExchange.Redis* dans le démarrage rapide. Vous avez également déjà configuré le paramètre d’application *CacheConnection* à utiliser localement et avec le service d’application publié. Utilisez cette même bibliothèque cliente et les informations *CacheConnection* contenues dans le *TeamsController* .

1. Dans l’ **Explorateur de solutions** , développez le dossier **Contrôleurs** et double-cliquez sur **TeamsController.cs** pour l’ouvrir.

    ![Contrôleur Teams](./media/cache-web-app-cache-aside-leaderboard/cache-teamscontroller.png)

1. Ajoutez les deux instructions `using` suivantes à **TeamsController.cs** :

    ```csharp
    using System.Configuration;
    using StackExchange.Redis;
    ```

1. Ajoutez les deux propriétés suivantes à la classe `TeamsController` :

    ```csharp
    // Redis Connection string info
    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
        return ConnectionMultiplexer.Connect(cacheConnection);
    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }
    ```

### <a name="update-the-teamscontroller-to-read-from-the-cache-or-the-database"></a>Mettre à jour les contrôleurs Teams pour lire à partir du cache ou de la base de données

Dans cet exemple, les statistiques d’équipe peuvent être récupérées à partir de la base de données ou à partir du cache. Les statistiques d’équipe sont stockées dans le cache comme `List<Team>`sérialisé et comme ensemble trié à l’aide des types de données Redis. Lors de la récupération des éléments d’un ensemble trié, vous pouvez récupérer certains éléments, récupérer tous les éléments ou effectuer une requête sur certains éléments. Dans cet exemple, vous allez interroger l’ensemble trié pour trouver les 5 meilleures équipes, classées par nombre de victoires.

Il n’est pas nécessaire de stocker les statistiques d’équipe dans plusieurs formats dans le cache pour utiliser Cache Azure pour Redis. Ce didacticiel utilise plusieurs formats pour illustrer certaines façons de mettre des données en cache et les différents types de données que vous pouvez utiliser à cette fin.

1. Ajoutez les instructions `using` suivantes au début du fichier `TeamsController.cs`, avec les autres instructions `using` :

    ```csharp
    using System.Diagnostics;
    using Newtonsoft.Json;
    ```

1. Remplacez la méthode d’implémentation `public ActionResult Index()` actuelle par l’implémentation suivante :

    ```csharp
    // GET: Teams
    public ActionResult Index(string actionType, string resultType)
    {
        List<Team> teams = null;

        switch(actionType)
        {
            case "playGames": // Play a new season of games.
                PlayGames();
                break;

            case "clearCache": // Clear the results from the cache.
                ClearCachedTeams();
                break;

            case "rebuildDB": // Rebuild the database with sample data.
                RebuildDB();
                break;
        }

        // Measure the time it takes to retrieve the results.
        Stopwatch sw = Stopwatch.StartNew();

        switch(resultType)
        {
            case "teamsSortedSet": // Retrieve teams from sorted set.
                teams = GetFromSortedSet();
                break;

            case "teamsSortedSetTop5": // Retrieve the top 5 teams from the sorted set.
                teams = GetFromSortedSetTop5();
                break;

            case "teamsList": // Retrieve teams from the cached List<Team>.
                teams = GetFromList();
                break;

            case "fromDB": // Retrieve results from the database.
            default:
                teams = GetFromDB();
                break;
        }

        sw.Stop();
        double ms = sw.ElapsedTicks / (Stopwatch.Frequency / (1000.0));

        // Add the elapsed time of the operation to the ViewBag.msg.
        ViewBag.msg += " MS: " + ms.ToString();

        return View(teams);
    }
    ```

1. Ajoutez les trois méthodes suivantes à la classe `TeamsController` pour implémenter les types d’action `playGames`, `clearCache` et `rebuildDB` de l’instruction switch ajoutée dans l’extrait de code précédent.

    La méthode `PlayGames` met à jour les statistiques d’équipe en simulant une saison de jeux, enregistre les résultats dans la base de données et efface les données désormais obsolètes à partir du cache.

    ```csharp
    void PlayGames()
    {
        ViewBag.msg += "Updating team statistics. ";
        // Play a "season" of games.
        var teams = from t in db.Teams
                    select t;

        Team.PlayGames(teams);

        db.SaveChanges();

        // Clear any cached results
        ClearCachedTeams();
    }
    ```

    La méthode `RebuildDB` réinitialise la base de données avec l’ensemble d’équipes par défaut, génère des statistiques pour ces équipes et efface les données désormais obsolètes à partir du cache.

    ```csharp
    void RebuildDB()
    {
        ViewBag.msg += "Rebuilding DB. ";
        // Delete and re-initialize the database with sample data.
        db.Database.Delete();
        db.Database.Initialize(true);

        // Clear any cached results
        ClearCachedTeams();
    }
    ```

    La méthode `ClearCachedTeams` supprime du cache toutes les statistiques d’équipe mises en cache.

    ```csharp
    void ClearCachedTeams()
    {
        IDatabase cache = Connection.GetDatabase();
        cache.KeyDelete("teamsList");
        cache.KeyDelete("teamsSortedSet");
        ViewBag.msg += "Team data removed from cache. ";
    }
    ```

1. Ajoutez les quatre méthodes suivantes à la classe `TeamsController` pour implémenter les différentes façons de récupérer les statistiques d’équipe à partir du cache et de la base de données. Chacune de ces méthodes retourne un `List<Team>` qui est ensuite affiché par la vue.

    La méthode `GetFromDB` lit les statistiques d’équipe à partir de la base de données.
    ```csharp
    List<Team> GetFromDB()
    {
        ViewBag.msg += "Results read from DB. ";
        var results = from t in db.Teams
            orderby t.Wins descending
            select t;

        return results.ToList<Team>();
    }
    ```

    La méthode `GetFromList` lit les statistiques d’équipe à partir du cache en tant que `List<Team>` sérialisé. Si les statistiques ne sont pas présentes dans le cache, une absence dans le cache se produit. Pour une absence dans le cache, les statistiques d’équipe sont lues à partir de la base de données, puis stockées dans le cache pour la prochaine requête. Dans cet exemple, la sérialisation JSON.NET est utilisée pour sérialiser les objets .NET vers et depuis le cache. Pour plus d’informations, consultez [Guide pratique pour utiliser des objets .NET dans Cache Azure pour Redis](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache).

    ```csharp
    List<Team> GetFromList()
    {
        List<Team> teams = null;

        IDatabase cache = Connection.GetDatabase();
        string serializedTeams = cache.StringGet("teamsList");
        if (!String.IsNullOrEmpty(serializedTeams))
        {
            teams = JsonConvert.DeserializeObject<List<Team>>(serializedTeams);

            ViewBag.msg += "List read from cache. ";
        }
        else
        {
            ViewBag.msg += "Teams list cache miss. ";
            // Get from database and store in cache
            teams = GetFromDB();

            ViewBag.msg += "Storing results to cache. ";
            cache.StringSet("teamsList", JsonConvert.SerializeObject(teams));
        }
        return teams;
    }
    ```

    La méthode `GetFromSortedSet` lit les statistiques d’équipe à partir d’un ensemble trié en cache. En cas d’absence dans le cache, les statistiques d’équipe sont lues à partir de la base de données, puis stockées dans le cache en tant qu’ensemble trié.

    ```csharp
    List<Team> GetFromSortedSet()
    {
        List<Team> teams = null;
        IDatabase cache = Connection.GetDatabase();
        // If the key teamsSortedSet is not present, this method returns a 0 length collection.
        var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", order: Order.Descending);
        if (teamsSortedSet.Count() > 0)
        {
            ViewBag.msg += "Reading sorted set from cache. ";
            teams = new List<Team>();
            foreach (var t in teamsSortedSet)
            {
                Team tt = JsonConvert.DeserializeObject<Team>(t.Element);
                teams.Add(tt);
            }
        }
        else
        {
            ViewBag.msg += "Teams sorted set cache miss. ";

            // Read from DB
            teams = GetFromDB();

            ViewBag.msg += "Storing results to cache. ";
            foreach (var t in teams)
            {
                Console.WriteLine("Adding to sorted set: {0} - {1}", t.Name, t.Wins);
                cache.SortedSetAdd("teamsSortedSet", JsonConvert.SerializeObject(t), t.Wins);
            }
        }
        return teams;
    }
    ```

    La méthode `GetFromSortedSetTop5` lit les cinq meilleures équipes à partir de l’ensemble trié en cache. Elle commence par vérifier l’existence de la clé `teamsSortedSet` dans le cache. Si cette clé n’est pas présente, la méthode `GetFromSortedSet` est appelée pour lire les statistiques d’équipe et les stocker dans le cache. Ensuite, l’ensemble trié mis en cache est interrogé de façon à trouver les cinq meilleures équipes, qui sont retournées.

    ```csharp
    List<Team> GetFromSortedSetTop5()
    {
        List<Team> teams = null;
        IDatabase cache = Connection.GetDatabase();

        // If the key teamsSortedSet is not present, this method returns a 0 length collection.
        var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
        if(teamsSortedSet.Count() == 0)
        {
            // Load the entire sorted set into the cache.
            GetFromSortedSet();

            // Retrieve the top 5 teams.
            teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
        }

        ViewBag.msg += "Retrieving top 5 teams from cache. ";
        // Get the top 5 teams from the sorted set
        teams = new List<Team>();
        foreach (var team in teamsSortedSet)
        {
            teams.Add(JsonConvert.DeserializeObject<Team>(team.Element));
        }
        return teams;
    }
    ```

### <a name="update-the-create-edit-and-delete-methods-to-work-with-the-cache"></a>Mettre à jour les méthodes Create, Edit et Delete pour utiliser le cache

Le code de génération de modèles automatique qui a été généré dans le cadre de cet exemple inclut des méthodes permettant d’ajouter, modifier et supprimer des équipes. Chaque fois qu’une équipe est ajoutée, modifiée ou supprimée, les données dans le cache deviennent obsolètes. Dans cette section, vous allez modifier ces trois méthodes pour effacer les équipes en cache, de sorte que le cache soit actualisé.

1. Accédez à la méthode `Create(Team team)` dans la classe `TeamsController`. Ajoutez un appel à la méthode `ClearCachedTeams`, comme indiqué dans l’exemple suivant :

    ```csharp
    // POST: Teams/Create
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Create([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
    {
        if (ModelState.IsValid)
        {
            db.Teams.Add(team);
            db.SaveChanges();
            // When a team is added, the cache is out of date.
            // Clear the cached teams.
            ClearCachedTeams();
            return RedirectToAction("Index");
        }

        return View(team);
    }
    ```

2. Accédez à la méthode `Edit(Team team)` dans la classe `TeamsController`. Ajoutez un appel à la méthode `ClearCachedTeams`, comme indiqué dans l’exemple suivant :

    ```csharp
    // POST: Teams/Edit/5
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Edit([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
    {
        if (ModelState.IsValid)
        {
            db.Entry(team).State = EntityState.Modified;
            db.SaveChanges();
            // When a team is edited, the cache is out of date.
            // Clear the cached teams.
            ClearCachedTeams();
            return RedirectToAction("Index");
        }
        return View(team);
    }
    ```

3. Accédez à la méthode `DeleteConfirmed(int id)` dans la classe `TeamsController`. Ajoutez un appel à la méthode `ClearCachedTeams`, comme indiqué dans l’exemple suivant :

    ```csharp
    // POST: Teams/Delete/5
    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public ActionResult DeleteConfirmed(int id)
    {
        Team team = db.Teams.Find(id);
        db.Teams.Remove(team);
        db.SaveChanges();
        // When a team is deleted, the cache is out of date.
        // Clear the cached teams.
        ClearCachedTeams();
        return RedirectToAction("Index");
    }
    ```

### <a name="add-caching-methods-to-the-teams-index-view"></a>Ajouter des méthodes de mise en cache à la vue Index des équipes

1. Dans l’ **Explorateur de solutions** , développez le dossier **Vues** puis le dossier **Équipes** , et double-cliquez sur **Index.cshtml** .

    ![Index.cshtml](./media/cache-web-app-cache-aside-leaderboard/cache-views-teams-index-cshtml.png)

1. Au début du fichier, recherchez l’élément de paragraphe suivant :

    ![Table d’actions](./media/cache-web-app-cache-aside-leaderboard/cache-teams-index-table.png)

    Ce lien crée une nouvelle équipe. Remplacez l’élément de paragraphe par la table suivante. Cette table contient des liens d’action pour créer une nouvelle équipe, jouer une nouvelle saison de jeux, effacer le cache, récupérer les équipes à partir du cache dans plusieurs formats, récupérer les équipes à partir de la base de données et reconstruire la base de données avec de nouvelles données d’exemple.

    ```html
    <table class="table">
        <tr>
            <td>
                @Html.ActionLink("Create New", "Create")
            </td>
            <td>
                @Html.ActionLink("Play Season", "Index", new { actionType = "playGames" })
            </td>
            <td>
                @Html.ActionLink("Clear Cache", "Index", new { actionType = "clearCache" })
            </td>
            <td>
                @Html.ActionLink("List from Cache", "Index", new { resultType = "teamsList" })
            </td>
            <td>
                @Html.ActionLink("Sorted Set from Cache", "Index", new { resultType = "teamsSortedSet" })
            </td>
            <td>
                @Html.ActionLink("Top 5 Teams from Cache", "Index", new { resultType = "teamsSortedSetTop5" })
            </td>
            <td>
                @Html.ActionLink("Load from DB", "Index", new { resultType = "fromDB" })
            </td>
            <td>
                @Html.ActionLink("Rebuild DB", "Index", new { actionType = "rebuildDB" })
            </td>
        </tr>
    </table>
    ```

1. Faites défiler le fichier **Index.cshtml** vers le bas pour visualiser la fin du fichier, puis ajoutez l’élément `tr` suivant, de sorte qu’il représente la dernière ligne de la dernière table du fichier :

    ```html
    <tr><td colspan="5">@ViewBag.Msg</td></tr>
    ```
    Cette ligne affiche la valeur `ViewBag.Msg` qui contient un rapport d’état sur l’opération en cours. La valeur `ViewBag.Msg` est définie lorsque vous cliquez sur l’un des liens d’action à l’étape précédente.

    ![Message d’état](./media/cache-web-app-cache-aside-leaderboard/cache-status-message.png)

1. Appuyez sur **F6** pour générer le projet.

## <a name="run-the-app-locally"></a>Exécutez l’application localement.

Exécutez l’application localement sur votre machine pour vérifier la fonctionnalité ajoutée pour prendre en charge les équipes.

Dans ce test, l’application et les bases de données s’exécutent localement. Toutefois, Cache Azure pour Redis est hébergé à distance dans Azure. Par conséquent, le cache réduira probablement les performances de la base de données. Pour de meilleures performances, l’application cliente et l’instance de Cache Azure pour Redis doivent se trouver au même emplacement. Dans la section suivante, vous allez déployer toutes les ressources sur Azure pour voir l’amélioration des performances avec l’utilisation d’un cache.

Pour exécuter l’application localement :

1. Appuyez sur **Ctrl+F5** pour exécuter l’application.

    ![Application s’exécutant localement](./media/cache-web-app-cache-aside-leaderboard/cache-local-application.png)

1. Testez chacune des nouvelles méthodes ajoutées à la vue. Étant donné que le cache est distant dans ces tests, la base de données doit subir une légère baisse de performances du cache.

## <a name="publish-and-run-in-azure"></a>Publier et exécuter l’application dans Azure

### <a name="provision-a-database-for-the-app"></a>Provisionner une base de données pour l’application

Dans cette section, vous allez provisionner une nouvelle base de données dans SQL Database que l’application utilisera pendant son hébergement dans Azure.

1. Dans le [portail Azure](https://portal.azure.com/), cliquez sur **Créer une ressource** dans le coin en haut à gauche du portail Azure.

1. Sur la page **Nouveau** , cliquez sur **Bases de données** > **Base de données SQL** .

1. Utilisez les paramètres suivants pour la nouvelle base de données SQL :

   | Paramètre       | Valeur suggérée | Description |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Nom de la base de données** | *ContosoTeamsDatabase* | Pour les noms de base de données valides, consultez [Database Identifiers](/sql/relational-databases/databases/database-identifiers) (Identificateurs de base de données). |
   | **Abonnement** | *Votre abonnement*  | Sélectionnez l’abonnement que vous avez utilisé pour créer le cache et héberger le service d’application. |
   | **Groupe de ressources**  | *TestResourceGroup* | Cliquez sur **Utiliser l’existant** et utilisez le groupe de ressources où vous avez placé votre cache et le service d’applications. |
   | **Sélectionner une source** | **Base de données vide** | Commencez avec une base de données vide. |

1. Sous **Serveur** , cliquez sur **Configurer les paramètres requis** > **Créer un nouveau serveur** et fournissez les informations suivantes, puis cliquez sur le bouton **Sélectionner** :

   | Paramètre       | Valeur suggérée | Description |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Nom du serveur** | Nom globalement unique | Pour les noms de serveur valides, consultez [Naming conventions](/azure/architecture/best-practices/resource-naming) (Conventions d’affectation de nom). |
   | **Connexion d’administrateur du serveur** | Nom valide | Pour les noms de connexion valides, consultez [Database Identifiers](/sql/relational-databases/databases/database-identifiers) (Identificateurs de base de données). |
   | **Mot de passe** | Mot de passe valide | Votre mot de passe doit comporter au moins 8 caractères et contenir des caractères appartenant à trois des catégories suivantes : caractères en majuscules, caractères en minuscules, chiffres et caractères non alphanumériques. |
   | **Lieu** | *USA Est* | Sélectionnez la région où vous avez créée le cache et le service d’applications. |

1. Cliquez sur **Épingler au tableau de bord** puis sur **Créer** pour créer la nouvelle base de données et le nouveau serveur.

1. Une fois la nouvelle base de données créée, cliquez sur **Afficher les chaînes de connexion de la base de données** et copiez la chaîne de connexion **ADO.NET** .

    ![Afficher les chaînes de connexion](./media/cache-web-app-cache-aside-leaderboard/cache-show-connection-strings.png)

1. Dans le portail Azure, accédez à votre service d’applications et cliquez sur **Paramètres d’application** , puis **Ajouter une nouvelle chaîne de connexion** sous la section des chaînes de connexion.

1. Ajouter une nouvelle chaîne de connexion nommée *TeamContext* pour correspondre à la classe de contexte de base de données Entity Framework. Collez la chaîne de connexion de votre nouvelle base de données comme valeur. Veillez à remplacer les espaces réservés suivants dans la chaîne de connexion et cliquez sur **Enregistrer** :

    | Espace réservé | Valeur suggérée |
    | --- | --- |
    | *{your_username}* | Utilisez les **informations de connexion d’administrateur de serveur** pour le serveur que vous venez de créer. |
    | *{your_password}* | Utilisez le mot de passe pour le serveur que vous venez de créer. |

    En ajoutant le nom d’utilisateur et un mot de passe comme paramètre d’application, votre nom d’utilisateur et un mot de passe ne sont pas inclus dans votre code. Cette approche permet de protéger les informations d’identification.

### <a name="publish-the-application-updates-to-azure"></a>Publier les mises à jour d'application dans Azure

Dans cette étape du didacticiel, vous allez publier les mises à jour de l’application sur Azure pour l’exécuter dans le cloud.

1. Cliquez avec le bouton droit sur le projet **ContosoTeamStats** dans Visual Studio, puis choisissez **Publier** .

    ![Publish](./media/cache-web-app-cache-aside-leaderboard/cache-publish-app.png)

2. Cliquez sur **Publier** pour utiliser le profil de publication que vous avez créé dans le démarrage rapide.

3. Une fois la publication terminée, Visual Studio lance l’application dans votre navigateur par défaut.

    ![Cache ajouté](./media/cache-web-app-cache-aside-leaderboard/cache-added-to-application.png)

    Le tableau suivant décrit chaque lien d’action de l’exemple d’application :

    | Action | Description |
    | --- | --- |
    | Création |Crée une équipe. |
    | Play Season |Joue une saison de jeux, met à jour les statistiques d’équipe et efface les données d’équipe obsolètes du cache. |
    | Clear Cache |Efface les statistiques d’équipe du cache. |
    | List from Cache |Récupère les statistiques d’équipe à partir cache. En cas d’absence dans le cache, charge les statistiques à partir de la base de données et les enregistre dans le cache pour la prochaine fois. |
    | Sorted Set from Cache |Récupère les statistiques d’équipe du cache à l’aide d’un ensemble trié. En cas d’absence dans le cache, charge les statistiques à partir de la base de données et les enregistre dans le cache à l’aide d’un ensemble trié. |
    | Top 5 Teams from Cache |Récupère les 5 meilleures équipes à partir du cache à l’aide d’un ensemble trié. En cas d’absence dans le cache, charge les statistiques à partir de la base de données et les enregistre dans le cache à l’aide d’un ensemble trié. |
    | Load from DB |Récupère les statistiques d’équipe à partir de la base de données. |
    | Rebuild DB |Reconstruit la base de données et la recharge avec les exemples de données d’équipe. |
    | Edit / Details / Delete |Modifie une équipe, affiche les détails d’une équipe, supprime une équipe. |

Cliquez sur certaines actions et essayez de récupérer les données de différentes sources. Notez que le temps nécessaire pour récupérer les données à partir de la base de données et du cache varie selon la méthode utilisée.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous avez terminé avec l’exemple d’application du didacticiel, vous pouvez supprimer les ressources Azure utilisées afin de réduire les coûts et de préserver les ressources. Toutes vos ressources doivent être contenues dans le même groupe de ressources, vous pouvez les supprimer ensemble en une seule opération en supprimant le groupe de ressources. Les instructions relatives à cette rubrique utilisaient un groupe de ressources nommé *TestResources* .

> [!IMPORTANT]
> La suppression d’un groupe de ressources est définitive ; le groupe de ressources et l’ensemble des ressources qu’il contient sont supprimés de manière permanente. Veillez à ne pas supprimer accidentellement des ressources ou un groupe de ressources incorrects. Si vous avez créé les ressources pour l’hébergement de cet exemple dans un groupe de ressources existant contenant des ressources que vous souhaitez conserver, vous pouvez supprimer chaque ressource individuellement à partir de son panneau respectif.
>

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et cliquez sur **Groupes de ressources** .
2. Tapez le nom de votre groupe de ressources dans la zone de texte **Filtrer des éléments...** .
3. Cliquez sur **...** à droite du votre groupe de ressources, puis cliquez sur **Supprimer le groupe de ressources** .

    ![DELETE](./media/cache-web-app-cache-aside-leaderboard/cache-delete-resource-group.png)

4. Il vous sera demandé de confirmer la suppression du groupe de ressources. Saisissez le nom de votre groupe de ressources pour confirmer, puis cliquez sur **Supprimer** .

    Après quelques instants, le groupe de ressources et toutes les ressources qu’il contient sont supprimés.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Mise à l’échelle de Cache Azure pour Redis](./cache-how-to-scale.md)