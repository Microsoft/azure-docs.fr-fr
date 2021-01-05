---
title: Superviser et diagnostiquer les services ASP.NET Core
description: Dans ce tutoriel, apprenez à configurer la supervision et les diagnostics pour une application ASP.NET Core Azure Service Fabric.
ms.topic: tutorial
ms.date: 07/10/2019
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: e7fe68c2d0c51ffcc67693da722d9243ea3506f7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91840793"
---
# <a name="tutorial-monitor-and-diagnose-an-aspnet-core-application-on-service-fabric-using-application-insights"></a>Tutoriel : Surveiller et diagnostiquer une application ASP.NET Core dans Service Fabric à l’aide d’Application Insights

Ce tutoriel est la cinquième partie de la série. Il passe en revue chacune des étapes nécessaires à la configuration de la supervision et des diagnostics pour une application ASP.NET Core s’exécutant sur un cluster Service Fabric en utilisant Application Insights. Nous collecterons les données de télémétrie à partir de l’application développée dans la première partie du didacticiel [Créer une application .NET Service Fabric](service-fabric-tutorial-create-dotnet-app.md).

Dans ce quatrième volet, vous apprenez à :
> [!div class="checklist"]
> * Configurer Application Insights pour votre application
> * Collecter les données de télémétrie de réponse pour effectuer le suivi de la communication basée sur HTTP entre les services
> * Utiliser la fonctionnalité de mappage des applications dans Application Insights
> * Ajouter des événements personnalisés à l’aide de l’API Application Insights

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Créer une application .NET Service Fabric](service-fabric-tutorial-create-dotnet-app.md)
> * [Déployer l’application sur un cluster distant](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * [Ajouter un point de terminaison HTTPS à un service frontal ASP.NET Core](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md)
> * [Configurer CI/CD à l’aide d’Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * Configurer la surveillance et les diagnostics pour l’application

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Installez Visual Studio 2019](https://www.visualstudio.com/) et les charges de travail **Développement Azure** et **Développement web et ASP.NET**.
* [Installez le Kit de développement logiciel (SDK) Service Fabric](service-fabric-get-started.md)

## <a name="download-the-voting-sample-application"></a>Télécharger l’exemple d’application de vote

Si vous n’avez pas généré l’exemple d’application de vote lors de la [première partie de cette série de didacticiels](service-fabric-tutorial-create-dotnet-app.md), vous pouvez le télécharger. Dans une fenêtre Commande ou sur un terminal, exécutez la commande suivante pour cloner le référentiel de l’exemple d’application sur votre ordinateur local.

```git
git clone https://github.com/Azure-Samples/service-fabric-dotnet-quickstart
```

## <a name="set-up-an-application-insights-resource"></a>Configurer une ressource Application Insights

Application Insights est la plateforme de gestion des performances des applications pour Azure et la plateforme recommandée par Service Fabric pour la surveillance et les diagnostics d’applications.

Pour créer une ressource Application Insights, accédez au [portail Azure](https://portal.azure.com). Sélectionnez **Créer une ressource** dans le menu de navigation gauche pour ouvrir la Place de marché Azure. Sélectionnez **Supervision et gestion**, puis **Application Insights**.

![Créer une ressource AI](./media/service-fabric-tutorial-monitoring-aspnet/new-ai-resource.png)

Vous devez à présent remplir les informations requises sur les attributs de la ressource à créer. Entrez un *nom*, un *groupe de ressources* et un *abonnement* appropriés. Sélectionnez *l’emplacement* où vous allez déployer votre cluster Service Fabric ultérieurement. Dans ce tutoriel, nous allons déployer l’application dans un cluster local, donc le champ *Emplacement* n’est pas requis. Le *Type d’application* doit être conservé sur « Application web ASP.NET »

![Attributs de ressource AI](./media/service-fabric-tutorial-monitoring-aspnet/new-ai-resource-attrib.png)

Une fois que vous avez fourni les informations nécessaires, sélectionnez **Créer** pour provisionner la ressource. Ce processus doit prendre environ une minute.
<!-- When completed, navigate to the newly deployed resource, and find the "Instrumentation Key" (visible in the "Essentials" drop down section). Copy it to clipboard, since we will need it in the next step. -->

## <a name="add-application-insights-to-the-applications-services"></a>Ajouter Application Insights aux services de l’application

Lancez Visual Studio 2019 avec des privilèges élevés en cliquant avec le bouton droit sur l’icône Visual Studio dans le menu Démarrer et en choisissant **Exécuter en tant qu’administrateur**. Sélectionnez **Fichier** > **Ouvrir** > **Projet/Solution** et accédez à l’application de vote (« Voting ») créée dans la première partie du tutoriel ou clonée à partir de git. Ouvrez *Voting.sln*. Si vous êtes invité à restaurer les packages NuGet de l’application, sélectionnez **Oui**.

Suivez ces étapes pour configurer Application Insights pour les services VotingWeb et VotingData :

1. Cliquez avec le bouton droit sur le nom du service, puis sélectionnez **Ajouter > Services connectés > Supervision avec Application Insights**.

    ![Configurer AI](./media/service-fabric-tutorial-monitoring-aspnet/configure-ai.png)
>[!NOTE]
>Selon le type de projet, quand vous cliquez avec le bouton droit sur le nom du service, vous pouvez avoir besoin de sélectionner Ajouter-> Application Insights Telemetry.

2. Sélectionnez **Prise en main**.
3. Connectez-vous au compte dont vous vous servez pour votre abonnement Azure et sélectionnez l’abonnement dans lequel vous avez créé la ressource Application Insights. Recherchez la ressource sous *Ressource Application Insights existante* dans la liste déroulante « Ressource ». Sélectionnez **Inscrire** pour ajouter Application Insights à votre service.

    ![Inscrire AI](./media/service-fabric-tutorial-monitoring-aspnet/register-ai.png)

4. Cliquez sur **Terminer** une fois que la boîte de dialogue qui s’affiche indique que l’action est terminée.

> [!NOTE]
> Veillez à effectuer les étapes ci-dessus pour **les deux** services dans l’application afin de terminer la configuration d’Application Insights pour l’application.
> La même ressource Application Insights est utilisée pour les deux services afin d’afficher les demandes entrantes et sortantes et la communication entre les services.

## <a name="add-the-microsoftapplicationinsightsservicefabricnative-nuget-to-the-services"></a>Ajouter le package NuGet Microsoft.ApplicationInsights.ServiceFabric.Native aux services

Application Insights a deux packages NuGet spécifiques à Service Fabric qui peuvent être utilisés selon le scénario. L’un est utilisé avec les services natifs de Service Fabric et l’autre avec les conteneurs et les exécutables invités. Dans ce cas, nous utiliserons le package NuGet Microsoft.ApplicationInsights.ServiceFabric.Native pour tirer parti de la présentation du contexte de service qu’il offre. Pour en savoir plus sur le kit SDK Application Insights et les packages NuGet propres à Service Fabric, consultez [Microsoft Application Insights pour Service Fabric](https://github.com/Microsoft/ApplicationInsights-ServiceFabric/blob/master/README.md).

Voici les étapes de configuration du package NuGet :

1. Cliquez avec le bouton droit sur la **solution de vote** (« Voting ») en haut de l’Explorateur de solutions, puis sélectionnez **Gérer les packages NuGet pour la solution...**
2. Sélectionnez **Parcourir** dans le menu de navigation en haut de la fenêtre « NuGet – Solution » et cochez la case **Inclure la préversion** en regard de la barre de recherche.
>[!NOTE]
>Vous devrez peut-être installer le package Microsoft.ServiceFabric.Diagnostics.Internal de la même manière s’il n’est pas préinstallé avant d’installer le package Application Insights.

3. Recherchez `Microsoft.ApplicationInsights.ServiceFabric.Native` et sélectionnez le package NuGet approprié.
4. Sur la droite, cochez les deux cases en regard des deux services dans l’application, **VotingWeb** et **VotingData**, puis sélectionnez **Installer**.
    ![NuGet du Kit de développement AI](./media/service-fabric-tutorial-monitoring-aspnet/ai-sdk-nuget-new.png)
5. Sélectionnez **OK** dans la boîte de dialogue *Aperçu des modifications* qui apparaît, puis *acceptez la licence*. Ceci termine l’ajout du package NuGet aux services.
6. Vous devez maintenant configurer l’initialiseur de télémétrie dans les deux services. Pour effectuer cette opération, ouvrez *VotingWeb.cs* et *VotingData.cs*. Pour les deux, exécutez la procédure suivante :
    1. Ajoutez ces deux instructions *using* en haut de chaque fichier *\<ServiceName>.cs*, après les instructions *using* existantes :

    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.ServiceFabric;
    ```

    2. Dans les deux fichiers, dans l’instruction *return* imbriquée de *CreateServiceInstanceListeners()* ou *CreateServiceReplicaListeners()* , sous *ConfigureServices* > *services*, avec les autres services Singleton déclarés, ajoutez :
    ```csharp
    .AddSingleton<ITelemetryInitializer>((serviceProvider) => FabricTelemetryInitializerExtension.CreateFabricTelemetryInitializer(serviceContext))
    ```
    Cette opération ajoute le *Contexte de service* à vos données de télémétrie, ce qui vous permet de mieux comprendre la source de votre télémétrie dans Application Insights. Votre instruction *return* imbriquée dans *VotingWeb.cs* doit ressembler à ceci :

    ```csharp
    return new WebHostBuilder()
        .UseKestrel()
        .ConfigureServices(
            services => services
                .AddSingleton<HttpClient>(new HttpClient())
                .AddSingleton<FabricClient>(new FabricClient())
                .AddSingleton<StatelessServiceContext>(serviceContext)
                .AddSingleton<ITelemetryInitializer>((serviceProvider) => FabricTelemetryInitializerExtension.CreateFabricTelemetryInitializer(serviceContext)))
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseStartup<Startup>()
        .UseApplicationInsights()
        .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
        .UseUrls(url)
        .Build();
    ```

    De même, dans *VotingData.cs*, vous devriez avoir :

    ```csharp
    return new WebHostBuilder()
        .UseKestrel()
        .ConfigureServices(
            services => services
                .AddSingleton<StatefulServiceContext>(serviceContext)
                .AddSingleton<IReliableStateManager>(this.StateManager)
                .AddSingleton<ITelemetryInitializer>((serviceProvider) => FabricTelemetryInitializerExtension.CreateFabricTelemetryInitializer(serviceContext)))
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseStartup<Startup>()
        .UseApplicationInsights()
        .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
        .UseUrls(url)
        .Build();
    ```

Vérifiez bien que la méthode `UseApplicationInsights()` est appelée dans les deux fichiers *VotingWeb.cs* et *VotingData.cs*, comme indiqué ci-dessus.

>[!NOTE]
>Cet exemple d’application utilise le protocole HTTP pour la communication des services. Si vous développez une application avec une communication à distance du service V2, vous devez également ajouter les lignes de code suivantes au même endroit que précédemment

```csharp
ConfigureServices(services => services
    ...
    .AddSingleton<ITelemetryModule>(new ServiceRemotingDependencyTrackingTelemetryModule())
    .AddSingleton<ITelemetryModule>(new ServiceRemotingRequestTrackingTelemetryModule())
)
```

À ce stade, vous êtes prêt à déployer l’application. Sélectionnez **Démarrer** en haut (ou **F5**) pour que Visual Studio génère et empaquette l’application, configure votre cluster local et y déploie l’application.

>[!NOTE]
>Vous pouvez obtenir une erreur de build si vous n’avez pas une version à jour du SDK .NET Core installé.

Une fois le déploiement de l’application terminé, accédez à `localhost:8080`, où vous devriez voir l’exemple d’application monopage de vote. Votez pour quelques éléments de votre choix afin de créer des exemples de données et de télémétrie. Personnellement, j’ai choisi les desserts !

![Exemples de votes AI](./media/service-fabric-tutorial-monitoring-aspnet/vote-sample.png)

N’hésitez pas également à *supprimer* certaines options de vote lorsque vous avez terminé d’ajouter quelques votes.

## <a name="view-telemetry-and-the-app-map-in-application-insights"></a>Afficher les données de télémétrie et la mise en correspondance d’applications dans Application Insights

Accédez à votre ressource Application Insights dans le portail Azure.

Sélectionnez **Vue d’ensemble** pour revenir à la page de destination de votre ressource. Sélectionnez ensuite **Recherche** en haut pour voir les traces entrantes. L’affichage des traces dans Application Insights prend quelques minutes. Si aucune trace ne s’affiche, patientez une minute et cliquez sur le bouton **Actualiser** en haut.
![Afficher les traces AI](./media/service-fabric-tutorial-monitoring-aspnet/ai-search.png)

Faites défiler la fenêtre *Recherche* vers le bas pour afficher toutes les données de télémétrie entrantes que vous obtenez instantanément avec Application Insights. Pour chaque action que vous avez effectuée dans l’application de vote, il doit exister une demande PUT sortante à partir de *VotingWeb* (Votes PUT/Put [nom]), une demande PUT entrante à partir de *VotingData* (PUT VoteData/Put [nom]), suivies d’une paire de demandes GET pour l’actualisation des données affichées. Il doit également y avoir une trace de dépendance pour HTTP sur localhost, car il s’agit de demandes HTTP. Voici un exemple de ce que vous verrez sur la façon dont un vote est ajouté :

![Exemple de trace de demande IA](./media/service-fabric-tutorial-monitoring-aspnet/sample-request.png)

Vous pouvez sélectionner une des traces pour afficher des informations complémentaires sur celle-ci. Application Insights fournit des informations utiles sur la demande, notamment le *temps de réponse* et *l’URL de la demande*. En outre, étant donné que vous avez ajouté le package NuGet spécifique à Service Fabric, vous obtiendrez également des données relatives à votre application dans le contexte d’un cluster Service Fabric dans la section *Données personnalisées* en dessous. Ces données incluent le contexte du service, afin de voir les éléments *PartitionID* et *ReplicaId* de la source de la demande et des problèmes mieux ciblés lors du diagnostic des erreurs dans votre application.

![Détails de la trace AI](./media/service-fabric-tutorial-monitoring-aspnet/trace-details.png)

Par ailleurs, vous pouvez sélectionner *Mise en correspondance d’applications* dans le menu de gauche de la page Vue d’ensemble, ou sélectionnez l’icône **Mise en correspondance d’applications** pour accéder à la mise en correspondance d’applications où figurent  affichant vos deux services connectés.

![Capture d’écran mettant en évidence Mise en correspondance d’applications dans le menu de gauche](./media/service-fabric-tutorial-monitoring-aspnet/app-map-new.png)

La mise en correspondance d’applications peut vous aider à mieux comprendre la topologie de votre application, en particulier si vous commencez à ajouter plusieurs services différents qui fonctionnent ensemble. Elle vous fournit également des données de base sur les taux de réussite des demandes et peut vous aider à diagnostiquer les demandes ayant échoué pour comprendre où se situe le problème. Pour en savoir plus sur la mise en correspondance d’applications, consultez [Mise en correspondance d’applications dans Application Insights](../azure-monitor/app/app-map.md).

## <a name="add-custom-instrumentation-to-your-application"></a>Ajouter une instrumentation personnalisée à votre application

Bien qu’Application Insights fournisse un grand nombre de données de télémétrie prêtes à l’emploi, vous voudrez peut-être ajouter une instrumentation personnalisée. Ceci peut être dû aux besoins de votre entreprise ou pour améliorer les diagnostics lorsque des problèmes apparaissent dans votre application. Application Insights a une API pour ingérer des événements et des métriques personnalisés, sur laquelle vous pouvez en apprendre plus [ici](../azure-monitor/app/api-custom-events-metrics.md).

Ajoutons des événements personnalisés à *VoteDataController.cs* (sous *VotingData* > *Contrôleurs*) pour effectuer le suivi lorsque des votes sont ajoutés et supprimés à partir de l’élément *votesDictionary* sous-jacent.

1. Ajoutez `using Microsoft.ApplicationInsights;` à la fin de l’autre à l’aide d’instructions.
2. Déclarez un nouveau *TelemetryClient* au début de la classe, sous la création de *IReliableStateManager* : `private TelemetryClient telemetry = new TelemetryClient();`.
3. Dans la fonction *Put()* , ajoutez un événement qui confirme qu’un vote a été ajouté. Ajoutez `telemetry.TrackEvent($"Added a vote for {name}");` une fois la transaction terminée, juste avant l’instruction *OkResult* retournée.
4. Dans *Delete()* , il existe un élément « if/else » basé sur la condition que *votesDictionary* contient des votes pour une option de vote donnée.
    1. Ajoutez un événement qui confirme la suppression d’un vote dans l’instruction *if*, après *await tx.CommitAsync()* : `telemetry.TrackEvent($"Deleted votes for {name}");`
    2. Ajoutez un événement pour indiquer que la suppression n’a pas eu lieu dans l’instruction *else*, avant l’instruction retournée : `telemetry.TrackEvent($"Unable to delete votes for {name}, voting option not found");`

Voici un exemple de ce à quoi vos fonctions *Put()* et *Delete()* peuvent ressembler après l’ajout des événements :

```csharp
// PUT api/VoteData/name
[HttpPut("{name}")]
public async Task<IActionResult> Put(string name)
{
    var votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

    using (ITransaction tx = this.stateManager.CreateTransaction())
    {
        await votesDictionary.AddOrUpdateAsync(tx, name, 1, (key, oldvalue) => oldvalue + 1);
        await tx.CommitAsync();
    }

    telemetry.TrackEvent($"Added a vote for {name}");
    return new OkResult();
}

// DELETE api/VoteData/name
[HttpDelete("{name}")]
public async Task<IActionResult> Delete(string name)
{
    var votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

    using (ITransaction tx = this.stateManager.CreateTransaction())
    {
        if (await votesDictionary.ContainsKeyAsync(tx, name))
        {
            await votesDictionary.TryRemoveAsync(tx, name);
            await tx.CommitAsync();
            telemetry.TrackEvent($"Deleted votes for {name}");
            return new OkResult();
        }
        else
        {
            telemetry.TrackEvent($"Unable to delete votes for {name}, voting option not found");
            return new NotFoundResult();
        }
    }
}
```

Une fois ces modifications effectuées, **démarrez** l’application pour qu’elle génère et déploie la dernière version. Une fois le déploiement de l’application terminé, accédez à `localhost:8080`, puis ajoutez et supprimez des options de vote. Ensuite, revenez à votre ressource Application Insights pour voir les traces de la dernière exécution (comme auparavant, l’affichage des traces dans Application Insights peut prendre entre une et deux minutes). Pour tous les votes que vous avez ajoutés et supprimés, vous devriez maintenant voir un « Événement personnalisé*, ainsi que toutes les données de télémétrie de réponse.

![événements personnalisés](./media/service-fabric-tutorial-monitoring-aspnet/custom-events.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :
> [!div class="checklist"]
> * Configurer Application Insights pour votre application
> * Collecter les données de télémétrie de réponse pour effectuer le suivi de la communication basée sur HTTP entre les services
> * Utiliser la fonctionnalité de mappage des applications dans Application Insights
> * Ajouter des événements personnalisés à l’aide de l’API Application Insights

Maintenant que vous avez configuré la supervision et les diagnostics pour votre application ASP.NET, essayez ce qui suit :

* [Explorer plus en détail la surveillance et les diagnostics dans Service Fabric](service-fabric-diagnostics-overview.md)
* [Analyse d’événement Service Fabric avec Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* Pour en savoir plus sur Application Insights, consultez [Documentation Application Insights](/azure/application-insights/)
