---
title: Application Insights pour les services cloud Azure | Microsoft Docs
description: Surveillance efficace de vos rôles Web et de travail avec Application Insights
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 09/05/2018
ms.openlocfilehash: ccd863db55ef0ff9f4051947321321c8b01430c4
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96920676"
---
# <a name="application-insights-for-azure-cloud-services"></a>Application Insights pour les services cloud Azure
[Application Insights][start] peut superviser les [applications de service cloud Azure](https://azure.microsoft.com/services/cloud-services/) pour vérifier la disponibilité, les performances, les échecs et l’utilisation en combinant les données des SDK Application Insights avec les données d’[Azure Diagnostics](../platform/diagnostics-extension-overview.md) de vos services cloud. Avec les retours que vous obtenez sur les performances et l’efficacité de votre application dans la nature, vous pouvez prendre des décisions avisées sur la direction de la conception de chaque cycle de développement.

![Vue d’ensemble du tableau de bord](./media/cloudservices/overview-graphs.png)

## <a name="prerequisites"></a>Prérequis
Avant de commencer, vous avez besoin des éléments suivants :

* Un abonnement [Azure](https://azure.com). Une connexion avec un compte Microsoft pour Windows, Xbox Live ou d’autres services cloud de Microsoft. 
* Microsoft Azure Tools 2.9 ou ultérieur.
* Developer Analytics Tools 7.10 ou ultérieur.

## <a name="get-started-quickly"></a>Prise en main rapide
Le moyen le plus simple et le plus rapide pour surveiller votre service cloud avec Application Insights consiste à choisir cette option quand vous publiez votre service dans Azure.

![Exemple de page Paramètres de diagnostic](./media/cloudservices/azure-cloud-application-insights.png)

Cette option instrumente votre application au moment de l’exécution, ce qui vous donne toutes les données de télémétrie dont vous avez besoin pour superviser les demandes, les exceptions et les dépendances dans votre rôle web. Elle supervise également les compteurs de performances de vos rôles de travail. Les traces de diagnostics générées par votre application sont également envoyées à Application Insights.

Si cette option vous suffit, vous avez terminé. 

Les étapes suivantes sont la [consultation des métriques de votre application](../platform/metrics-charts.md), l’[interrogation de vos données avec Analytics](../log-query/log-query-overview.md). 

Pour superviser les performances dans le navigateur, vous pouvez configurer des [tests de disponibilité](./monitor-web-app-availability.md) et [ajouter du code à vos pages web](./javascript.md).

Les sections suivantes décrivent les options supplémentaires suivantes :

* Envoyer des données de différents composants et différentes configurations de build à des ressources distinctes.
* Ajouter une télémétrie personnalisée à partir de votre application.

## <a name="sample-app-instrumented-with-application-insights"></a>Exemple d’application instrumentée avec Application Insights
Dans cet [exemple d’application](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService), Application Insights est ajouté à un service cloud avec deux rôles de travail hébergés dans Azure. 

Dans la section suivante, vous découvrez comment adapter votre propre projet de service cloud de la même façon.

## <a name="plan-resources-and-resource-groups"></a>Planifier des ressources et des groupes de ressources
Les données de télémétrie de votre application sont stockées, analysées et affichées dans une ressource Azure de type Application Insights. 

Chaque ressource appartient à un groupe de ressources. Les groupes de ressources servent à gérer les coûts, à accorder l’accès aux membres d’équipe et à déployer des mises à jour dans une seule transaction coordonnée. Par exemple, vous pouvez [écrire un script pour déployer](../../azure-resource-manager/templates/deploy-powershell.md) un service cloud Azure et ses ressources de supervision Application Insights en une seule opération.

### <a name="resources-for-components"></a>Ressources pour les composants
Nous vous recommandons de créer une ressource distincte pour chaque composant de votre application. Autrement dit, vous créez une ressource pour chaque rôle web et rôle de travail. Vous pouvez analyser chaque composant séparément, mais vous créez un [tableau de bord](./overview-dashboard.md) qui réunit les principaux graphiques de tous les composants, pour pouvoir les comparer et les superviser ensemble dans une même vue. 

Une autre approche consiste à envoyer les données de télémétrie de plusieurs rôles à la même ressource, mais en [ajoutant une propriété de dimension à chaque élément de télémétrie](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) qui identifie son rôle source. Dans cette approche, les graphiques de métriques comme les exceptions affichent normalement une agrégation des décomptes des différents rôles, mais vous pouvez segmenter le graphique en fonction de l’identificateur de rôle si nécessaire. Vous pouvez également filtrer les recherches sur la même dimension. Cette solution facilite légèrement la consultation simultanée de tous les éléments, mais elle peut aussi entraîner une certaine confusion entre les rôles.

La télémétrie de navigateur est généralement comprise dans la même ressource que son rôle web côté serveur.

Placez les ressources Application Insights pour les différents composants dans un groupe de ressources. De cette façon, vous pouvez facilement les gérer ensemble. 

### <a name="separate-development-test-and-production"></a>Séparer développement, test et production
Si vous développez des événements personnalisés pour votre prochaine fonctionnalité alors que la version précédente est active, vous devez envoyer la télémétrie de développement à une ressource Application Insights distincte. Sinon, vous retrouvez difficilement vos données de télémétrie de test dans l’ensemble du trafic provenant du site actif.

Pour éviter cette situation, créez des ressources distinctes pour chaque configuration de build ou « tampon » (développement, test, production, etc.) de votre système. Placez les ressources pour chaque configuration de build dans un groupe de ressources distinct. 

Pour envoyer les données de télémétrie aux ressources appropriées, vous pouvez configurer le SDK Application Insights pour qu’il récupère une clé d’instrumentation différente en fonction de la configuration de build. 

Découvrez comment [définir dynamiquement la clé d’instrumentation](./separate-resources.md#dynamic-ikey) pour différentes étapes. 

## <a name="create-an-application-insights-resource-for-each-role"></a>Création d’une ressource Application Insights pour chaque rôle

Si vous avez décidé de créer une ressource distincte pour chaque rôle (et éventuellement un ensemble distinct pour chaque configuration de build), créez-les plutôt dans le portail Application Insights. Si vous créez beaucoup de ressources, vous pouvez [automatiser le processus](./powershell.md).

1. Dans le [portail Azure][portal], sélectionnez **Nouveau** > **Services de développement** > **Application Insights**.  

    ![Volet Application Insights](./media/cloudservices/01-new.png)

1. Dans la liste déroulante **Type d’application**, sélectionnez **Application web ASP.NET**.

Chaque ressource est identifiée par une clé d’instrumentation. Vous pouvez avoir besoin de cette clé plus tard pour configurer ou vérifier manuellement la configuration du SDK.


## <a name="set-up-azure-diagnostics-for-each-role"></a>Configurer Diagnostics Azure pour chaque rôle
Définissez cette option pour surveiller votre application avec Application Insights. Pour les rôles web, cette option permet de superviser les performances, de recevoir des alertes, des diagnostics et d’analyser l’utilisation. Pour d’autres rôles, vous pouvez rechercher et superviser des diagnostics Azure comme le redémarrage, les compteurs de performances et les appels à System.Diagnostics.Trace. 

1. Dans l’Explorateur de solutions Visual Studio, sous **\<YourCloudService>**  > **Rôles**, ouvrez les propriétés de chaque rôle.

1. Dans **Configuration**, cochez la case **Envoyer des données de diagnostic à Application Insights** et sélectionnez la ressource Application Insights que vous avez créée plus tôt.

Si vous avez décidé d’utiliser une ressource Application Insights distincte pour chaque configuration de build, sélectionnez d’abord la configuration.

![Configurer Application Insights](./media/cloudservices/configure-azure-diagnostics.png)

L’objectif est d’insérer vos clés d’instrumentation Application Insights dans les fichiers nommés *ServiceConfiguration.\*.cscfg*. Voici l’[exemple de code](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/AzureEmailService/ServiceConfiguration.Cloud.cscfg).

Si vous voulez changer le niveau des informations de diagnostics envoyées à Application Insights, [modifiez directement les fichiers *.cscfg*](../platform/diagnostics-extension-to-application-insights.md).

## <a name="install-the-sdk-in-each-project"></a><a name="sdk"></a>Installation du Kit de développement logiciel (SDK) dans chaque projet
Avec cette option, vous pouvez ajouter une télémétrie métier personnalisée à n’importe quel rôle. L’option permet d’analyser en détail l’utilisation et les performances de votre application.

Dans Visual Studio, configurez le Kit de développement logiciel (SDK) Application Insights pour chaque projet d’application cloud.

1. Pour configurer des **rôles web**, cliquez avec le bouton droit sur le projet et sélectionnez **Configurer Application Insights** ou **Ajouter > Télémétrie Application Insights**.

1. Pour configurer des **rôles de travail** : 

    a. Cliquez avec le bouton droit sur le projet et sélectionnez **Gérer les packages NuGet**.

    b. Ajoutez [Application Insights pour les serveurs Windows](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/).

1. Pour configurer le SDK afin d’envoyer des données à la ressource Application Insights :

    a. Dans une fonction de démarrage appropriée, définissez la clé d’instrumentation à partir du paramètre de configuration dans le fichier *.cscfg* :
 
    ```csharp
        TelemetryConfiguration configuration = TelemetryConfiguration.CreateDefault();
        configuration.InstrumentationKey = RoleEnvironment.GetConfigurationSettingValue("APPINSIGHTS_INSTRUMENTATIONKEY");
        var telemetryClient = new TelemetryClient(configuration);
    ```
   
    b. Répétez l’« étape a » pour chaque rôle de votre application. Consultez les exemples :
   
    * [Rôle Web](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Global.asax.cs#L27)
    * [Rôle de travail](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L232)
    * [Pour les pages web](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Views/Shared/_Layout.cshtml#L13) 

1. Définissez le fichier *ApplicationInsights.config* pour qu’il soit toujours copié dans le répertoire de sortie.

   Un message dans le fichier *.config* vous demande de placer ici la clé d’instrumentation. Toutefois, pour les applications cloud, définissez-la à partir du fichier *.cscfg*. De cette façon, le rôle est identifié correctement dans le portail.

## <a name="set-up-status-monitor-to-collect-full-sql-queries-optional"></a>Configurer Status Monitor pour collecter des requêtes SQL complètes (facultatif)

Cette étape est uniquement nécessaire si vous voulez capturer des requêtes SQL complètes sur .NET Framework. 

1. Dans le fichier `\*.csdef`, ajoutez une [tâche de démarrage](../../cloud-services/cloud-services-startup-tasks.md) pour chaque rôle similaire à 

    ```xml
    <Startup>
      <Task commandLine="AppInsightsAgent\InstallAgent.bat" executionContext="elevated" taskType="simple">
        <Environment>
          <Variable name="ApplicationInsightsAgent.DownloadLink" value="http://go.microsoft.com/fwlink/?LinkID=522371" />
          <Variable name="RoleEnvironment.IsEmulated">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
    ```
    
2. Téléchargez [InstallAgent.bat](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/AppInsightsAgent/InstallAgent.bat) et [InstallAgent.ps1](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/AppInsightsAgent/InstallAgent.ps1), et placez-les dans le dossier `AppInsightsAgent` de chaque projet de rôle. Veillez à les copier dans le répertoire de sortie par le biais des propriétés de fichier Visual Studio ou de scripts de build.

3. Sur tous les workers, ajoutez les variables d’environnement : 

    ```xml
      <Environment>
        <Variable name="COR_ENABLE_PROFILING" value="1" />
        <Variable name="COR_PROFILER" value="{324F817A-7420-4E6D-B3C1-143FBED6D855}" />
        <Variable name="MicrosoftInstrumentationEngine_Host" value="{CA487940-57D2-10BF-11B2-A3AD5A13CBC0}" />
      </Environment>
    ```
    
## <a name="run-and-publish-the-app"></a>Exécution et publication de l’application

1. Exécutez votre application et connectez-vous à Azure. 

1. Ouvrez la ressource Application Insights que vous avez créée.

   Les points de données individuels sont affichés dans [Recherche][diagnostic] et les données agrégées sont affichées dans [Metrics Explorer](../platform/metrics-charts.md).

1. Ajoutez plus de données de télémétrie (voir les sections suivantes) et publiez votre application pour obtenir en direct des diagnostics et un retour sur l’utilisation. 

S’il n’y a pas de données, effectuez ce qui suit :

1. Pour voir les événements individuels, ouvrez la vignette [Rechercher][diagnostic].
1. Dans l’application, ouvrez différentes pages pour générer des données de télémétrie.
1. Attendez quelques secondes, puis cliquez sur **Actualiser**.  

Pour plus d’informations, consultez la page [Dépannage][qna].

## <a name="view-azure-diagnostics-events"></a>Voir les événements Diagnostics Azure
Les informations d’[Azure Diagnostics](../platform/diagnostics-extension-overview.md) sont disponibles dans Application Insights aux emplacements suivants :

* Les compteurs de performances s’affichent comme mesures personnalisées. 
* Les journaux des événements Windows s’affichent comme traces et événements personnalisés.
* Les journaux des applications, les journaux d’activité ETW et tous les journaux d’activité d’infrastructure des diagnostics s’affichent comme traces.

Pour voir les compteurs de performances et le nombre d’événements, ouvrez [Metrics Explorer](../platform/metrics-charts.md) et ajoutez le graphique suivant :

![Données Azure Diagnostics](./media/cloudservices/23-wad.png)

Pour rechercher dans les différents journaux de trace envoyés par Azure Diagnostics, utilisez [Recherche](./diagnostic-search.md) ou une [requête Analytics](../log-query/log-analytics-tutorial.md). Par exemple, supposons qu’une exception non prise en charge a provoqué le blocage et le recyclage d’un rôle. Cette information s’affiche dans le canal Application du Journal des événements Windows. Vous pouvez utiliser Recherche pour voir l’erreur du Journal des événements Windows et obtenir la trace complète de l’exception. De cette façon, vous pouvez identifier la cause racine du problème.

![Recherche de Diagnostics Azure](./media/cloudservices/25-wad.png)

## <a name="more-telemetry"></a>Données de télémétrie supplémentaires
Les sections suivantes décrivent comment obtenir davantage de données de télémétrie à partir de différents aspects de votre application.

## <a name="track-requests-from-worker-roles"></a>Suivre les demandes des rôles de travail
Dans les rôles Web, le module des requêtes collecte automatiquement les données relatives aux requêtes HTTP. Pour obtenir des exemples de remplacement du comportement de collecte par défaut, consultez l’[exemple MVCWebRole](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/MvcWebRole). 

Vous pouvez capturer les performances des appels aux rôles de travail en effectuant le suivi de la même façon que les requêtes HTTP. Dans Application Insights, le type de demande de télémétrie mesure une unité de travail côté serveur nommée, pouvant être chronométrée et réussir ou échouer indépendamment. Même si les requêtes HTTP sont automatiquement capturées par le SDK, vous pouvez insérer votre propre code pour suivre les requêtes envoyées aux rôles de travail.

Consultez les deux exemples de rôles de travail instrumentés pour signaler des requêtes : 
* [WorkerRoleA](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/WorkerRoleA)
* [WorkerRoleB](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/WorkerRoleB)

## <a name="exceptions"></a>Exceptions
Pour plus d’informations sur la collecte des exceptions non prises en charge à partir de différents types d’application web, consultez [Supervision des exceptions dans Application Insights](./asp-net-exceptions.md).

L’exemple du rôle Web dispose de contrôleurs MVC5 et API Web 2. Les exceptions non gérées à partir de deux sont capturées avec les gestionnaires suivants :

* [AiHandleErrorAttribute](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Telemetry/AiHandleErrorAttribute.cs) configuré pour les contrôleurs MVC5 [comme indiqué dans cet exemple](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/App_Start/FilterConfig.cs#L12) 
* [AiWebApiExceptionLogger](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Telemetry/AiWebApiExceptionLogger.cs) configuré pour les contrôleurs de l’API Web 2 [comme indiqué dans cet exemple](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/App_Start/WebApiConfig.cs#L25) 

Pour les rôles de travail, il existe deux façons de suivre les exceptions :

* Utilisez TrackException(ex).
* Si vous avez ajouté le package NuGet de l’écouteur de trace Application Insights, vous pouvez utiliser System.Diagnostics.Trace pour journaliser les exceptions [comme indiqué dans cet exemple](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L107).

## <a name="performance-counters"></a>Compteurs de performance
Les compteurs suivants sont collectés par défaut :

* \Processus(??APP_WIN32_PROC??)\% Temps processeur
* \Memory\Octets disponibles
* \.Exceptions .NET CLR(??APP_CLR_PROC??)\# Nombre d'exceptions levées/s
* \Processus(??APP_WIN32_PROC??)\Octets privés
* \Processus(??APP_WIN32_PROC??)\Nombre d’octets de données E/S par s
* \Processor(_Total)\% temps processeur

Pour les rôles web, ces compteurs sont également collectés :

* \Applications ASP.NET(??APP_W3SVC_PROC??)\Demandes/s
* \Applications ASP.NET (??APP_W3SVC_PROC??)\Durée d’exécution de la demande
* \Applications ASP.NET (??APP_W3SVC_PROC??)\Demandes dans la file d’attente d’application

Vous pouvez spécifier des compteurs de performances personnalisés ou d’autres compteurs de performances Windows en modifiant *ApplicationInsights.config* [comme indiqué dans cet exemple](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/ApplicationInsights.config#L14).

  ![Compteurs de performance](./media/cloudservices/002-servers.png)

## <a name="correlated-telemetry-for-worker-roles"></a>Télémétrie corrélée pour les rôles de travail
Pour un diagnostic complet, vous pouvez voir la cause de l’échec d’une demande ou d’une latence élevée. Avec les rôles web, le SDK configure automatiquement une corrélation entre les données de télémétries associées. 

Pour avoir la même chose avec les rôles de travail, vous pouvez utiliser un initialiseur de télémétrie personnalisé pour définir un attribut de contexte Operation.Id commun à toutes les télémétries. De cette façon, vous voyez en un coup d’œil si le problème de latence ou l’échec est lié à une dépendance ou à votre code. 

Voici comment faire :

* Définissez l’ID de corrélation dans un CallContext [comme indiqué dans cet exemple](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L36). Dans ce cas, nous utilisons l’ID de demande comme ID de corrélation.
* Ajoutez une implémentation personnalisée de TelemetryInitializer pour définir l’Operation.Id sur le correlationId défini précédemment. Pour obtenir un exemple, consultez [ItemCorrelationTelemetryInitializer](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/Telemetry/ItemCorrelationTelemetryInitializer.cs#L13).
* Ajoutez l'initialiseur de télémétrie personnalisée. Pour ce faire, utilisez le fichier *ApplicationInsights.config* ou du code [comme indiqué dans cet exemple](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L233).

## <a name="client-telemetry"></a>Télémétrie client
Pour obtenir des données de télémétrie basées sur un navigateur, comme le nombre de pages consultées, le temps de chargement des pages ou les exceptions de script, et pour écrire des données de télémétrie personnalisées dans vos scripts de page, consultez [Ajouter le SDK JavaScript à vos pages web][client].

## <a name="availability-tests"></a>Tests de disponibilité
Pour vérifier que votre application est en ligne et réactive, [Configurez des tests web][availability].

## <a name="display-everything-together"></a>Afficher tous les éléments ensemble
Pour une vue d’ensemble de votre système, vous pouvez rassembler les principaux graphiques de supervision sur un même [tableau de bord](./overview-dashboard.md). Vous pouvez par exemple épingler le décompte des demandes et des échecs de chaque rôle. 

Si votre système utilise d’autres services Azure, comme Stream Analytics, ajoutez également leurs graphiques de supervision. 

Si vous avez une application mobile cliente, utilisez [App Center](../learn/mobile-center-quickstart.md). Créez des requêtes dans [Analytics](../log-query/log-query-overview.md) pour afficher le nombre d’événements, et épinglez-les au tableau de bord.

## <a name="example"></a>Exemple
[L'exemple](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService) analyse un service qui dispose d’un rôle Web et de deux rôles de travail.

## <a name="exception-method-not-found-on-running-in-azure-cloud-services"></a>Exception « méthode introuvable » pendant l’exécution dans les services cloud Azure
Avez-vous effectué une génération pour .NET 4.6 ? .NET 4.6 n’est pas automatiquement pris en charge dans les rôles de services cloud Azure. [Installez .NET 4.6 sur chaque rôle](../../cloud-services/cloud-services-dotnet-install-dotnet.md) avant d’exécuter votre application.

## <a name="video"></a>Vidéo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="next-steps"></a>Étapes suivantes
* [Configuration de l’envoi de diagnostics Azure à Application Insights](../platform/diagnostics-extension-to-application-insights.md)
* [Créer automatiquement des ressources Application Insights](./powershell.md)
* [Automatiser les diagnostics Azure](./powershell-azure-diagnostics.md)
* [Azure Functions](https://github.com/christopheranderson/azure-functions-app-insights-sample)

[api]: ./api-custom-events-metrics.md
[availability]: ./monitor-web-app-availability.md
[azure]: ./app-insights-overview.md
[client]: ./javascript.md
[diagnostic]: ./diagnostic-search.md
[netlogs]: ./asp-net-trace-logs.md
[portal]: https://portal.azure.com/
[qna]: ../faq.md
[redfield]: ./monitor-performance-live-website-now.md
[start]: ./app-insights-overview.md