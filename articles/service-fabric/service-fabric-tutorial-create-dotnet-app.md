---
title: Créer une application .NET sur Service Fabric dans Azure
description: Dans ce tutoriel, vous découvrez comment créer une application avec un service frontal ASP.NET Core et un service principal fiable avec état, et déployer l’application sur un cluster.
ms.topic: tutorial
ms.date: 07/10/2019
ms.custom: mvc, devx-track-js, devx-track-csharp
ms.openlocfilehash: a783f5338dbc7ce0832b346492490525da332e39
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96297115"
---
# <a name="tutorial-create-and-deploy-an-application-with-an-aspnet-core-web-api-front-end-service-and-a-stateful-back-end-service"></a>Tutoriel : Créer et déployer une application avec un service frontal API Web ASP.NET Core et un service principal avec état

Ce tutoriel est la première partie d’une série d’étapes.  Vous allez découvrir comment créer une application Azure Service Fabric avec un service frontal API Web ASP.NET Core et un service principal avec état pour stocker vos données. Lorsque vous avez terminé, vous disposez d’une application de vote avec un composant web frontal ASP.NET Core qui enregistre les résultats de vote dans un service principal avec état dans le cluster. Cette série de tutoriels demande un ordinateur de développement Windows. Si vous ne souhaitez pas créer l’application de vote manuellement, vous pouvez [télécharger le code source](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/) pour obtenir l’application terminée et passer directement au [Guide de l’exemple d’application de vote](#walkthrough_anchor).  Si vous préférez, vous pouvez également regarder une [vidéo de procédure pas-à-pas](https://channel9.msdn.com/Events/Connect/2017/E100) de ce tutoriel.

![API front-end AngularJS+ASP.NET - Connexion à un service back-end avec état dans Service Fabric](./media/service-fabric-tutorial-create-dotnet-app/application-diagram.png)

Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]
> * Créer un service API Web ASP.NET Core en tant que service fiable avec état
> * Créer un service d’application Web ASP.NET Core en tant que service web sans état
> * Utiliser le proxy inverse pour communiquer avec le service avec état

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Créer une application .NET Service Fabric
> * [Déployer l’application sur un cluster distant](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * [Ajouter un point de terminaison HTTPS à un service frontal ASP.NET Core](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md)
> * [Configurer CI/CD à l’aide d’Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-monitoring-aspnet.md)

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :
* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Installez Visual Studio 2019](https://www.visualstudio.com/) version 15.5 ou ultérieure avec les charges de travail **Développement Azure** et **Développement web et ASP.NET**.
* [Installez le Kit de développement logiciel (SDK) Service Fabric](service-fabric-get-started.md)

## <a name="create-an-aspnet-web-api-service-as-a-reliable-service"></a>Créer un service API Web ASP.NET en tant que service fiable

Commencez par créer le web frontal de l’application de vote à l’aide d’ASP.NET Core. ASP.NET Core est une infrastructure légère de développement web inter-plateformes, que vous pouvez utiliser pour créer des API web et d’interfaces utilisateur web modernes. Pour bien comprendre comment ASP.NET Core s’intègre avec Service Fabric, il est vivement recommandé de lire l’article [ASP.NET Core dans le modèle Reliable Services de Service Fabric](service-fabric-reliable-services-communication-aspnetcore.md). Pour l’instant, vous pouvez suivre ce tutoriel pour démarrer rapidement. Pour en savoir plus sur ASP.NET Core, consultez la [Documentation d’ASP.NET Core](/aspnet/core/).

1. Lancez Visual Studio en tant qu’**administrateur**.

2. Créez un projet en sélectionnant **Fichier**->**Nouveau**->**Projet**.

3. Dans la boîte de dialogue **Nouveau projet**, sélectionnez **Cloud > Application Service Fabric**.

4. Nommez l’application **Voting**, puis cliquez sur **OK**.

   ![Boîte de dialogue Nouveau projet dans Visual Studio](./media/service-fabric-tutorial-create-dotnet-app/new-project-dialog.png)

5. Dans la page **Nouveau service Service Fabric**, choisissez **ASP.NET Core sans état** et nommez votre service **VotingWeb**, puis cliquez sur **OK**.
   
   ![Choix du service web ASP.NET dans la boîte de dialogue Nouveau service](./media/service-fabric-tutorial-create-dotnet-app/new-project-dialog-2.png) 

6. La page suivante fournit un ensemble de modèles de projets ASP.NET Core. Pour ce tutoriel, sélectionnez **Application web (Model-View-Controller)** , puis cliquez sur **OK**.
   
   ![Choisir le type de projet ASP.NET](./media/service-fabric-tutorial-create-dotnet-app/vs-new-aspnet-project-dialog.png)

   Visual Studio crée une application et un projet de service, et les affiche dans l’Explorateur de solutions.

   ![Explorateur de solutions après la création de l’application avec un service API Web ASP.NET Core]( ./media/service-fabric-tutorial-create-dotnet-app/solution-explorer-aspnetcore-service.png)

### <a name="update-the-sitejs-file"></a>Mettre à jour le fichier site.js
Ouvrez **wwwroot/js/site.js**.  Remplacez son contenu par le code JavaScript suivant utilisé par les vues d’accueil, puis enregistrez vos modifications.

```javascript
var app = angular.module('VotingApp', ['ui.bootstrap']);
app.run(function () { });

app.controller('VotingAppController', ['$rootScope', '$scope', '$http', '$timeout', function ($rootScope, $scope, $http, $timeout) {

    $scope.refresh = function () {
        $http.get('api/Votes?c=' + new Date().getTime())
            .then(function (data, status) {
                $scope.votes = data;
            }, function (data, status) {
                $scope.votes = undefined;
            });
    };

    $scope.remove = function (item) {
        $http.delete('api/Votes/' + item)
            .then(function (data, status) {
                $scope.refresh();
            })
    };

    $scope.add = function (item) {
        var fd = new FormData();
        fd.append('item', item);
        $http.put('api/Votes/' + item, fd, {
            transformRequest: angular.identity,
            headers: { 'Content-Type': undefined }
        })
            .then(function (data, status) {
                $scope.refresh();
                $scope.item = undefined;
            })
    };
}]);
```

### <a name="update-the-indexcshtml-file"></a>Mettre à jour le fichier Index.cshtml

Ouvrez le fichier **Views/Home/Index.cshtml**. Il s’agit de la vue spécifique au contrôleur Home.  Remplacez son contenu par la commande suivante, puis enregistrez vos modifications.

```html
@{
    ViewData["Title"] = "Service Fabric Voting Sample";
}

<div ng-controller="VotingAppController" ng-init="refresh()">
    <div class="container-fluid">
        <div class="row">
            <div class="col-xs-8 col-xs-offset-2 text-center">
                <h2>Service Fabric Voting Sample</h2>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <form class="col-xs-12 center-block">
                    <div class="col-xs-6 form-group">
                        <input id="txtAdd" type="text" class="form-control" placeholder="Add voting option" ng-model="item"/>
                    </div>
                    <button id="btnAdd" class="btn btn-default" ng-click="add(item)">
                        <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>
                        Add
                    </button>
                </form>
            </div>
        </div>

        <hr/>

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <div class="row">
                    <div class="col-xs-4">
                        Click to vote
                    </div>
                </div>
                <div class="row top-buffer" ng-repeat="vote in votes.data">
                    <div class="col-xs-8">
                        <button class="btn btn-success text-left btn-block" ng-click="add(vote.Key)">
                            <span class="pull-left">
                                {{vote.Key}}
                            </span>
                            <span class="badge pull-right">
                                {{vote.Value}} Votes
                            </span>
                        </button>
                    </div>
                    <div class="col-xs-4">
                        <button class="btn btn-danger pull-right btn-block" ng-click="remove(vote.Key)">
                            <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>
                            Remove
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

### <a name="update-the-_layoutcshtml-file"></a>Mettre à jour le fichier _Layout.cshtml

Ouvrez le fichier **Views/Shared/_Layout.cshtml**. Il s’agit de la disposition par défaut pour l’application ASP.NET.  Remplacez son contenu par la commande suivante, puis enregistrez vos modifications.


```html
<!DOCTYPE html>
<html ng-app="VotingApp" xmlns:ng="https://angularjs.org">
<head>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>@ViewData["Title"]</title>

    <link href="~/lib/bootstrap/dist/css/bootstrap.css" rel="stylesheet"/>
    <link href="~/css/site.css" rel="stylesheet"/>

</head>
<body>
<div class="container body-content">
    @RenderBody()
</div>

<script src="~/lib/jquery/dist/jquery.js"></script>
<script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.7.2/angular.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/2.5.0/ui-bootstrap-tpls.js"></script>
<script src="~/js/site.js"></script>

@RenderSection("Scripts", required: false)
</body>
</html>
```

### <a name="update-the-votingwebcs-file"></a>Mettre à jour le fichier VotingWeb.cs

Ouvrez le fichier *VotingWeb.cs*. Il crée le WebHost Core ASP.NET dans le service sans état à l’aide du serveur web WebListener.

Ajoutez la directive `using System.Net.Http;` au début du fichier.

Remplacez la fonction `CreateServiceInstanceListeners()` par la commande suivante, puis enregistrez vos modifications.

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(
            serviceContext =>
                new KestrelCommunicationListener(
                    serviceContext,
                    "ServiceEndpoint",
                    (url, listener) =>
                    {
                        ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

                        return new WebHostBuilder()
                            .UseKestrel()
                            .ConfigureServices(
                                services => services
                                    .AddSingleton<HttpClient>(new HttpClient())
                                    .AddSingleton<FabricClient>(new FabricClient())
                                    .AddSingleton<StatelessServiceContext>(serviceContext))
                            .UseContentRoot(Directory.GetCurrentDirectory())
                            .UseStartup<Startup>()
                            .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                            .UseUrls(url)
                            .Build();
                    }))
    };
}
```

Ajoutez également le code suivant `GetVotingDataServiceName` méthode ci-dessous `CreateServiceInstanceListeners()`, puis enregistrez vos modifications. `GetVotingDataServiceName` renvoie le nom du service lorsqu’il est interrogé.

```csharp
internal static Uri GetVotingDataServiceName(ServiceContext context)
{
    return new Uri($"{context.CodePackageActivationContext.ApplicationName}/VotingData");
}
```

### <a name="add-the-votescontrollercs-file"></a>Ajouter le fichier VotesController.cs

Ajoutez un contrôleur qui définit les actions de vote. Cliquez avec le bouton droit sur le dossier **Contrôleurs** et sélectionnez **Ajouter -> Nouvel élément -> Visual C# -> ASP.NET Core -> Classe**. Nommez le fichier **VotesController.cs** et cliquez sur **Ajouter**.  

Remplacez le contenu du fichier *VotesController.cs* par la commande suivante, puis enregistrez vos modifications.  Plus tard, dans [Mettre à jour le fichier VotesController.cs](#updatevotecontroller_anchor), ce fichier est modifié pour lire et écrire des données de vote à partir du service principal.  Pour l’instant, le contrôleur renvoie des données de chaîne statique à la vue.

```csharp
namespace VotingWeb.Controllers
{
    using System;
    using System.Collections.Generic;
    using System.Fabric;
    using System.Fabric.Query;
    using System.Linq;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using Newtonsoft.Json;

    [Produces("application/json")]
    [Route("api/Votes")]
    public class VotesController : Controller
    {
        private readonly HttpClient httpClient;

        public VotesController(HttpClient httpClient)
        {
            this.httpClient = httpClient;
        }

        // GET: api/Votes
        [HttpGet]
        public async Task<IActionResult> Get()
        {
            List<KeyValuePair<string, int>> votes= new List<KeyValuePair<string, int>>();
            votes.Add(new KeyValuePair<string, int>("Pizza", 3));
            votes.Add(new KeyValuePair<string, int>("Ice cream", 4));

            return Json(votes);
        }
     }
}
```

### <a name="configure-the-listening-port"></a>Configurer le port d’écoute

Lorsque le service frontal VotingWeb est créé, Visual Studio sélectionne de manière aléatoire le port d’écoute du service.  Le service VotingWeb joue le rôle de serveur frontal de cette application et accepte le trafic externe. Associez ce service à un port fixe et connu.  Le [manifeste de service](service-fabric-application-and-service-manifests.md) déclare les points de terminaison de service.

Dans l’Explorateur de solutions, ouvrez *VotingWeb/PackageRoot/ServiceManifest.xml*.  Recherchez l’élément **Endpoint** dans la section **Resources**, puis remplacez la valeur du **port** par **8080**. Pour déployer et exécuter l’application localement, le port d’écoute de l’application doit être ouvert et disponible sur votre ordinateur.

```xml
<Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8080" />
    </Endpoints>
  </Resources>
```

Mettez également à jour la valeur de la propriété d’URL d’application dans le projet Voting, afin qu’un navigateur web s’ouvre sur le port adéquat lorsque vous effectuez le débogage de votre application.  Dans l’Explorateur de solutions, sélectionnez le projet **Voting** et définissez la propriété **URL de l’application** sur **8080**.

### <a name="deploy-and-run-the-voting-application-locally"></a>Déployer et exécuter l’application Voting localement
Vous pouvez maintenant continuer et exécuter l’application pour le débogage. Dans Visual Studio, appuyez sur **F5** pour déployer l’application sur votre cluster Service Fabric local en mode débogage. L’application échoue si vous n’avez pas précédemment ouvert Visual Studio en tant qu’**administrateur**.

> [!NOTE]
> La première fois que vous exécutez et déployez l’application localement, Visual Studio crée un cluster Service Fabric local pour le débogage.  La création de cluster peut prendre un certain temps. L’état de la création du cluster est affiché dans la fenêtre Sortie Visual Studio.

Une fois que l’application de vote a été déployée sur votre cluster Service Fabric local, votre application web s’ouvre automatiquement dans un onglet de navigateur et doit ressembler à ceci :

![ASP.NET Core frontal](./media/service-fabric-tutorial-create-dotnet-app/debug-front-end.png)

Pour arrêter le débogage de l’application, revenez à Visual Studio et appuyez sur **Maj + F5**.

## <a name="add-a-stateful-back-end-service-to-your-application"></a>Ajouter un service principal avec état à votre application

Maintenant qu’un service API Web ASP.NET est exécuté dans l’application, ajoutez un service fiable avec état pour stocker des données dans l’application.

Service Fabric permet de stocker de manière cohérente et fiable vos données directement dans votre service à l’aide de collections fiables. Les collections fiables sont un ensemble de classes de collections hautement disponibles et fiables avec lesquelles tous ceux qui ont utilisé les collections C# sont familiers.

Dans ce tutoriel, vous créez un service qui stocke une valeur de compteur dans une collection fiable.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **Services** dans le projet d’application Voting et choisissez **Ajouter > Nouveau service Service Fabric**.
    
2. Dans la boîte de dialogue **Nouveau service Service Fabric**, choisissez **ASP.NET Core avec état**, nommez le service **VotingData**, puis appuyez sur **OK**.

    Une fois votre projet de service créé, votre application contient deux services. À mesure que vous continuez à développer votre application, vous pouvez ajouter d’autres services de la même façon. Chacun peut être mis à niveau et faire l'objet d'un contrôle de version indépendamment.

3. La page suivante fournit un ensemble de modèles de projets ASP.NET Core. Pour ce tutoriel, sélectionnez **API**.

    Visual Studio crée un projet de service VotingData et l’affiche dans l’Explorateur de solutions.

    ![Explorateur de solutions](./media/service-fabric-tutorial-create-dotnet-app/solution-explorer-aspnetcore-webapi-service.png)

### <a name="add-the-votedatacontrollercs-file"></a>Ajouter le fichier VoteDataController.cs

Dans le projet **VotingData**, cliquez avec le bouton droit sur le dossier **Contrôleurs** et sélectionnez **Ajouter -> Nouvel élément -> Classe**. Nommez le fichier **VoteDataController.cs** et cliquez sur **Ajouter**. Remplacez le contenu du fichier par la commande suivante, puis enregistrez vos modifications.

```csharp
namespace VotingData.Controllers
{
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.ServiceFabric.Data;
    using Microsoft.ServiceFabric.Data.Collections;

    [Route("api/[controller]")]
    public class VoteDataController : Controller
    {
        private readonly IReliableStateManager stateManager;

        public VoteDataController(IReliableStateManager stateManager)
        {
            this.stateManager = stateManager;
        }

        // GET api/VoteData
        [HttpGet]
        public async Task<IActionResult> Get()
        {
            CancellationToken ct = new CancellationToken();

            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                Microsoft.ServiceFabric.Data.IAsyncEnumerable<KeyValuePair<string, int>> list = await votesDictionary.CreateEnumerableAsync(tx);

                Microsoft.ServiceFabric.Data.IAsyncEnumerator<KeyValuePair<string, int>> enumerator = list.GetAsyncEnumerator();

                List<KeyValuePair<string, int>> result = new List<KeyValuePair<string, int>>();

                while (await enumerator.MoveNextAsync(ct))
                {
                    result.Add(enumerator.Current);
                }

                return this.Json(result);
            }
        }

        // PUT api/VoteData/name
        [HttpPut("{name}")]
        public async Task<IActionResult> Put(string name)
        {
            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                await votesDictionary.AddOrUpdateAsync(tx, name, 1, (key, oldvalue) => oldvalue + 1);
                await tx.CommitAsync();
            }

            return new OkResult();
        }

        // DELETE api/VoteData/name
        [HttpDelete("{name}")]
        public async Task<IActionResult> Delete(string name)
        {
            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                if (await votesDictionary.ContainsKeyAsync(tx, name))
                {
                    await votesDictionary.TryRemoveAsync(tx, name);
                    await tx.CommitAsync();
                    return new OkResult();
                }
                else
                {
                    return new NotFoundResult();
                }
            }
        }
    }
}
```

## <a name="connect-the-services"></a>Connecter les services

Dans l’étape qui suit, connectez les deux services et faites en sorte que l’application web frontale obtienne et définisse les informations de vote à partir du service principal.

Service Fabric fournit une flexibilité complète sur votre façon de communiquer avec Reliable Services. Dans une même application, vous pouvez avoir des services qui sont accessibles via TCP, d’autres services accessibles via une API REST HTTP et encore d’autres services accessibles via des sockets web. Pour obtenir des informations sur les options disponibles et leurs avantages/inconvénients respectifs, consultez [Communication avec les services](service-fabric-connect-and-communicate-with-services.md).

Dans ce tutoriel, utilisez l’[API web ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md) et le [proxy inverse Service Fabric](service-fabric-reverseproxy.md) pour que le service web frontal VotingWeb puisse communiquer avec le service de données principal VotingWeb. Le proxy inverse est configuré par défaut pour utiliser le port 19081 et doit fonctionner pour ce tutoriel. Le port du proxy inverse est défini dans le modèle Azure Resource Manager utilisé pour configurer le cluster. Pour connaître le port utilisé, examinez le modèle de cluster dans la ressource **Microsoft.ServiceFabric/clusters** : 

```json
"nodeTypes": [
          {
            ...
            "httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
            "isPrimary": true,
            "vmInstanceCount": "[parameters('nt0InstanceCount')]",
            "reverseProxyEndpointPort": "[parameters('SFReverseProxyPort')]"
          }
        ],
```
Pour rechercher le port de proxy inverse utilisé dans votre cluster de développement local, examinez l’élément **HttpApplicationGatewayEndpoint** dans le manifeste de cluster Service Fabric local :
1. Ouvrez une fenêtre de navigateur et accédez à http:\//localhost:19080 pour ouvrir l’outil Service Fabric Explorer.
2. Sélectionnez **Cluster -> Manifeste**.
3. Notez l’élément de port HttpApplicationGatewayEndpoint. Il doit s’agir par défaut de 19081. Si tel n’est pas le cas, vous devez modifier le port dans la méthode GetProxyAddress du code VotesController.cs suivant.

<a id="updatevotecontroller" name="updatevotecontroller_anchor"></a>

### <a name="update-the-votescontrollercs-file"></a>Mettre à jour le fichier VotesController.cs

Dans le projet **VotingWeb**, ouvrez le fichier *Controllers/VotesController.cs*.  Remplacez le contenu de définition de classe `VotesController` par la commande suivante, puis enregistrez vos modifications. Si le port de proxy inverse découvert à l’étape précédente n’est pas 19081, modifiez le port utilisé dans la méthode GetProxyAddress en remplaçant la valeur 19081 par celle du port que vous avez découvert.

```csharp
public class VotesController : Controller
{
    private readonly HttpClient httpClient;
    private readonly FabricClient fabricClient;
    private readonly StatelessServiceContext serviceContext;

    public VotesController(HttpClient httpClient, StatelessServiceContext context, FabricClient fabricClient)
    {
        this.fabricClient = fabricClient;
        this.httpClient = httpClient;
        this.serviceContext = context;
    }

    // GET: api/Votes
    [HttpGet("")]
    public async Task<IActionResult> Get()
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);

        ServicePartitionList partitions = await this.fabricClient.QueryManager.GetPartitionListAsync(serviceName);

        List<KeyValuePair<string, int>> result = new List<KeyValuePair<string, int>>();

        foreach (Partition partition in partitions)
        {
            string proxyUrl =
                $"{proxyAddress}/api/VoteData?PartitionKey={((Int64RangePartitionInformation) partition.PartitionInformation).LowKey}&PartitionKind=Int64Range";

            using (HttpResponseMessage response = await this.httpClient.GetAsync(proxyUrl))
            {
                if (response.StatusCode != System.Net.HttpStatusCode.OK)
                {
                    continue;
                }

                result.AddRange(JsonConvert.DeserializeObject<List<KeyValuePair<string, int>>>(await response.Content.ReadAsStringAsync()));
            }
        }

        return this.Json(result);
    }

    // PUT: api/Votes/name
    [HttpPut("{name}")]
    public async Task<IActionResult> Put(string name)
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);
        long partitionKey = this.GetPartitionKey(name);
        string proxyUrl = $"{proxyAddress}/api/VoteData/{name}?PartitionKey={partitionKey}&PartitionKind=Int64Range";

        StringContent putContent = new StringContent($"{{ 'name' : '{name}' }}", Encoding.UTF8, "application/json");
        putContent.Headers.ContentType = new MediaTypeHeaderValue("application/json");

        using (HttpResponseMessage response = await this.httpClient.PutAsync(proxyUrl, putContent))
        {
            return new ContentResult()
            {
                StatusCode = (int) response.StatusCode,
                Content = await response.Content.ReadAsStringAsync()
            };
        }
    }

    // DELETE: api/Votes/name
    [HttpDelete("{name}")]
    public async Task<IActionResult> Delete(string name)
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);
        long partitionKey = this.GetPartitionKey(name);
        string proxyUrl = $"{proxyAddress}/api/VoteData/{name}?PartitionKey={partitionKey}&PartitionKind=Int64Range";

        using (HttpResponseMessage response = await this.httpClient.DeleteAsync(proxyUrl))
        {
            if (response.StatusCode != System.Net.HttpStatusCode.OK)
            {
                return this.StatusCode((int) response.StatusCode);
            }
        }

        return new OkResult();
    }


    /// <summary>
    /// Constructs a reverse proxy URL for a given service.
    /// Example: http://localhost:19081/VotingApplication/VotingData/
    /// </summary>
    /// <param name="serviceName"></param>
    /// <returns></returns>
    private Uri GetProxyAddress(Uri serviceName)
    {
        return new Uri($"http://localhost:19081{serviceName.AbsolutePath}");
    }

    /// <summary>
    /// Creates a partition key from the given name.
    /// Uses the zero-based numeric position in the alphabet of the first letter of the name (0-25).
    /// </summary>
    /// <param name="name"></param>
    /// <returns></returns>
    private long GetPartitionKey(string name)
    {
        return Char.ToUpper(name.First()) - 'A';
    }
}
```

<a id="walkthrough" name="walkthrough_anchor"></a>

## <a name="walk-through-the-voting-sample-application"></a>Guide de l’exemple d’application de vote

L’application de vote se compose de deux services :

* Service frontal web (VotingWeb) : service frontal web ASP.NET Core qui fait office de page web et expose les API web pour communiquer avec le service principal.
* Service principal (VotingData) : service web ASP.NET Core qui expose une API pour stocker les résultats de vote dans un dictionnaire fiable rendu persistant sur disque.

![Diagramme de l’application](./media/service-fabric-tutorial-create-dotnet-app/application-diagram.png)

Lorsque vous votez dans l’application, les événements suivants se produisent :

1. Un JavaScript envoie la demande de vote à l’API web dans le service frontal web en tant que demande HTTP PUT.

2. Le service frontal web utilise un proxy pour localiser et transférer une demande HTTP PUT au service principal.

3. Le service principal accepte la demande entrante, puis stocke le résultat mis à jour dans un dictionnaire fiable, qui est répliqué sur plusieurs nœuds au sein du cluster et rendu persistant sur disque. Toutes les données de l’application étant stockées dans le cluster, aucune base de données n’est nécessaire.

## <a name="debug-in-visual-studio"></a>Déboguer dans Visual Studio

Lors du débogage d’application dans Visual Studio, vous utilisez un cluster de développement Service Fabric local. Vous avez la possibilité d’adapter votre expérience de débogage à votre scénario. Dans cette application, stockez les données dans le service principal à l’aide d’un dictionnaire fiable. Visual Studio supprime l’application par défaut lorsque vous arrêtez le débogueur. La suppression de l’application a pour effet de supprimer également les données dans le service principal. Pour rendre les données persistantes entre les sessions de débogage, vous pouvez modifier le **Mode de débogage de l'application** en tant que propriété sur le projet **Voting** dans Visual Studio.

Pour examiner ce qui se produit dans le code, procédez comme suit :

1. Ouvrez le fichier **VotingWeb\VotesController.cs** et définissez un point d’arrêt dans la méthode **Put** de l’API web (ligne 72).

2. Ouvrez le fichier **VotingData\VoteDataController.cs** et définissez un point d’arrêt dans la méthode **Put** de l’API web (ligne 54).

3. Appuyez sur **F5** pour exécuter l'application en mode débogage.

4. Revenez en arrière dans le navigateur, puis cliquez sur une option de vote ou ajoutez une nouvelle option de vote. Vous avez atteint le premier point d’arrêt dans le contrôleur d’api du service frontal web.
    

   1. Il s’agit de l’emplacement où le JavaScript dans le navigateur envoie une demande au contrôleur d’API web dans le service frontal.

      ![Ajouter un service de vote frontal](./media/service-fabric-tutorial-create-dotnet-app/addvote-frontend.png)

   2. Tout d’abord, créez l’URL du ReverseProxy pour le service principal **(1)** .
   3. Ensuite, envoyez la requête HTTP PUT au ReverseProxy **(2)** .
   4. Pour finir, renvoyez la réponse du service principal au client **(3)** .

5. Appuyez sur **F5** pour continuer.
   1. Vous êtes à présent au point d’arrêt dans le service principal.

      ![Ajouter un service de vote principal](./media/service-fabric-tutorial-create-dotnet-app/addvote-backend.png)

   2. Dans la première ligne de la méthode **(1)** , utilisez le `StateManager` pour obtenir ou ajouter un dictionnaire fiable nommé `counts`.
   3. Toutes les interactions avec des valeurs d’un dictionnaire fiable requièrent une transaction. Cette instruction using **(2)** crée cette transaction.
   4. Dans la transaction, mettez à jour la valeur de la clé appropriée pour l’option de vote et validez l’opération **(3)** . Lorsque la méthode commit retourne des données, celles-ci sont mises à jour dans le dictionnaire et répliquées sur d’autres nœuds du cluster. Les données sont à présent stockées en sécurité dans le cluster, et le service principal peut basculer vers d’autres nœuds, tout en gardant les données disponibles.
6. Appuyez sur **F5** pour continuer.

Pour arrêter la session de débogage, appuyez sur **Maj+F5**.

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Créer un service API Web ASP.NET Core en tant que service fiable avec état
> * Créer un service d’application Web ASP.NET Core en tant que service web sans état
> * Utiliser le proxy inverse pour communiquer avec le service avec état

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Déployer l’application dans Azure](service-fabric-tutorial-deploy-app-to-party-cluster.md)
