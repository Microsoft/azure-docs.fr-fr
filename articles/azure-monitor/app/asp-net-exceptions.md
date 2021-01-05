---
title: Diagnostiquer des défaillances et exceptions avec Azure Application Insights
description: Capturez des exceptions à partir d’applications ASP.NET, ainsi que des données de télémétrie des demandes.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 07/11/2019
ms.openlocfilehash: 36e916eabfca8e997fc3d46ff10f6201203457cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936501"
---
# <a name="diagnose-exceptions-in-your-web-apps-with-application-insights"></a>Diagnostiquez les exceptions dans vos applications web avec Application Insights
Les exceptions dans votre application web dynamique sont signalées par [Application Insights](./app-insights-overview.md). Vous pouvez associer les demandes ayant échoué à des exceptions et à d’autres événements sur le client et le serveur, ce qui vous permet de diagnostiquer rapidement les causes.

## <a name="set-up-exception-reporting"></a>Configurer les rapports d’exceptions
* Pour que les exceptions soient signalées par votre application de serveur :
  * Applications web Azure : Ajoutez [l’extension Application Insights](./azure-web-apps.md)
  * Applications hébergées par IIS sur une machine virtuelle Azure et un groupe de machines virtuelles identiques Azure : Ajoutez l’[extension Application Monitoring](./azure-vm-vmss-apps.md)
  * Installez le [SDK Application Insights](./asp-net.md) dans votre code d’application, ou
  * Serveurs web IIS : Exécutez l’[agent Application Insights](./monitor-performance-live-website-now.md) ou
  * Applications web Java : Activer [l’agent Java](./java-in-process-agent.md)
* Installez l’[extrait de code JavaScript](./javascript.md) dans vos pages web pour intercepter les exceptions du navigateur.
* Dans certains frameworks d’application ou avec certains paramètres, vous devez prendre des mesures supplémentaires pour intercepter davantage d’exceptions :
  * [Web forms](#web-forms)
  * [MVC](#mvc)
  * [API web 1.*](#web-api-1x)
  * [API web 2.*](#web-api-2x)
  * [WCF](#wcf)

  Cet article se concentre spécifiquement sur les applications .NET Framework du point de vue d’un exemple de code. Certaines des méthodes qui fonctionnent pour .NET Framework sont obsolètes dans le kit SDK .NET Core. Reportez-vous à la [documentation du kit SDK .NET Core ](./asp-net-core.md) si vous avez une application .NET Core.

## <a name="diagnosing-exceptions-using-visual-studio"></a>Diagnostic des exceptions à l’aide de Visual Studio
Ouvrez la solution d’application dans Visual Studio pour faciliter le débogage.

Exécutez l’application sur votre serveur ou sur votre ordinateur de développement à l’aide de la touche F5.

Ouvrez la fenêtre de recherche d’Application Insights dans Visual Studio et configurez-la pour afficher les événements depuis votre application. En cours de débogage, il vous suffit de cliquer sur le bouton Application Insights pour effectuer ce paramétrage.

![Cliquez avec le bouton droit sur le projet et sélectionnez Application Insights, Ouvrir.](./media/asp-net-exceptions/34.png)

Notez que vous pouvez filtrer le rapport pour qu’il affiche uniquement les exceptions.

*Aucune exception ne s’affiche ? Consultez [Capture des exceptions](#exceptions)*

Cliquez sur un rapport d’exception pour afficher sa trace de pile.
Cliquez sur une référence de ligne dans l’arborescence des appels de procédure pour ouvrir le fichier de code approprié.

Dans le code, notez que CodeLens affiche les données sur les exceptions :

![Notification CodeLens des exceptions.](./media/asp-net-exceptions/35.png)

## <a name="diagnosing-failures-using-the-azure-portal"></a>Diagnostic des défaillances à l’aide du portail Azure
Application Insights est fourni avec une expérience APM organisée pour vous aider à diagnostiquer les échecs dans les applications surveillées. Pour démarrer, cliquez sur l’option Échecs dans le menu de ressource Application Insights situé dans la section Examiner.
Une vue en plein écran apparaît et vous indique les tendances du taux d’échec pour vos demandes, le nombre d'entre elles qui échouent et le nombre d’utilisateurs qui en sont affectés. Sur la droite s’affichent certaines des distributions les plus utiles pour l’opération défaillante sélectionnée, avec les trois premiers codes de réponse, les trois premiers types d’exception et les trois premiers types de dépendances défaillantes.

![Vue de triage des échecs (onglet des opérations)](./media/asp-net-exceptions/failures0719.png)

Vous pouvez consulter en un seul clic des exemples représentatifs pour chacun de ces sous-ensembles d’opérations. En particulier, pour diagnostiquer les exceptions, vous pouvez cliquer sur le nombre d’exceptions pour une exception spécifique à présenter dans l’onglet Détails de la transaction de bout en bout, comme indiqué ci-dessous :

![Onglet Détails de la transaction de bout en bout](./media/asp-net-exceptions/end-to-end.png)

**Sinon**, au lieu de rechercher les exceptions relatives à une opération défaillante spécifique, vous pouvez partir de la vue d’ensemble des exceptions, en basculant vers l’onglet Exceptions en haut. Sur cette page, vous pouvez voir toutes les exceptions collectées pour votre application analysée.

*Aucune exception ne s’affiche ? Consultez [Capture des exceptions](#exceptions)*


## <a name="custom-tracing-and-log-data"></a>Suivi personnalisé et données du journal
Pour obtenir des données de diagnostic propres à votre application, vous pouvez insérer le code pour envoyer vos propres données de télémétrie. Ces informations apparaissent dans Recherche de diagnostic avec la requête, une vue de la page et d’autres données automatiquement collectées.

Vous disposez de plusieurs options :

* [TrackEvent()](./api-custom-events-metrics.md#trackevent) sert généralement à surveiller les modèles d’utilisation, mais les données qu’il envoie apparaissent également sous Evénements personnalisés dans Recherche de diagnostic. Les événements sont nommés et peuvent contenir des propriétés de type chaîne et des métriques numériques sur lesquels vous pouvez [filtrer vos recherches de diagnostic](./diagnostic-search.md).
* [TrackTrace()](./api-custom-events-metrics.md#tracktrace) vous permet d’envoyer des données plus longues telles que des informations POST.
* [TrackException()](#exceptions) envoie des arborescences des appels de procédure. [Plus d’informations sur les exceptions](#exceptions).
* Si vous utilisez déjà un framework de journalisation comme Log4Net ou NLog, vous pouvez [capturer ces journaux d’activité](asp-net-trace-logs.md) et les visualiser dans Recherche de diagnostic avec les données sur les demandes et les exceptions.

Pour voir ces événements, ouvrez [Recherche](./diagnostic-search.md) dans le menu de gauche, sélectionnez le menu déroulant **Types d’événements**, puis choisissez événement personnalisé, trace ou exception.

![Extraire](./media/asp-net-exceptions/customevents.png)

> [!NOTE]
> Si votre application génère un volume important de télémétrie, le module d'échantillonnage adaptatif réduit automatiquement le volume qui est envoyé vers le portail en envoyant uniquement une fraction représentative des événements. Les événements qui font partie de la même opération seront activés ou désactivés en tant que groupe, afin que vous puissiez naviguer entre les événements connexes. [En savoir plus sur l'échantillonnage.](./sampling.md)
>
>

### <a name="how-to-see-request-post-data"></a>Affichage des données POST de la demande
Les détails de la demande n'incluent pas les données envoyées à votre application dans un appel POST. Pour que ces données soient signalées :

* [Installez le SDK](./asp-net.md) dans votre projet d’application.
* Insérez du code dans votre application pour appeler [Microsoft.ApplicationInsights.TrackTrace()](./api-custom-events-metrics.md#tracktrace). Envoyez les données POST dans le paramètre du message. Il existe une limite à la taille autorisée. Vous pouvez donc essayer d'envoyer uniquement les données essentielles.
* Lorsque vous examinez une demande ayant échoué, recherchez les traces associées.

## <a name="capturing-exceptions-and-related-diagnostic-data"></a><a name="exceptions"></a> Capture des exceptions et des données de diagnostic connexes
Dans un premier temps, vous ne verrez pas dans le portail toutes les exceptions qui entraînent des défaillances dans votre application. Vous verrez les exceptions du navigateur (si vous utilisez le [SDK JavaScript](./javascript.md) dans vos pages web). Mais la plupart des exceptions de serveur sont interceptées par IIS et vous devez écrire un peu de code afin de les afficher.

Vous pouvez :

* **Enregistrer explicitement des exceptions** en insérant le code dans les gestionnaires d'exceptions pour signaler ces exceptions.
* **Capturer automatiquement des exceptions** en configurant votre infrastructure ASP.NET. Les ajouts nécessaires sont différents selon les différents types d’infrastructure.

## <a name="reporting-exceptions-explicitly"></a>Signalisation explicite des exceptions
La façon la plus simple consiste à insérer un appel à TrackException() dans un gestionnaire d'exceptions.

```javascript
    try
    { ...
    }
    catch (ex)
    {
      appInsights.trackException(ex, "handler loc",
        {Game: currentGame.Name,
         State: currentGame.State.ToString()});
    }
```

```csharp
    var telemetry = new TelemetryClient();
    ...
    try
    { ...
    }
    catch (Exception ex)
    {
       // Set up some properties:
       var properties = new Dictionary <string, string>
         {{"Game", currentGame.Name}};

       var measurements = new Dictionary <string, double>
         {{"Users", currentGame.Users.Count}};

       // Send the exception telemetry:
       telemetry.TrackException(ex, properties, measurements);
    }
```

```VB
    Dim telemetry = New TelemetryClient
    ...
    Try
      ...
    Catch ex as Exception
      ' Set up some properties:
      Dim properties = New Dictionary (Of String, String)
      properties.Add("Game", currentGame.Name)

      Dim measurements = New Dictionary (Of String, Double)
      measurements.Add("Users", currentGame.Users.Count)

      ' Send the exception telemetry:
      telemetry.TrackException(ex, properties, measurements)
    End Try
```

Les paramètres de propriétés et les mesures sont facultatifs, mais sont utiles pour [filtrer et ajouter](./diagnostic-search.md) des informations supplémentaires. Par exemple, si vous avez une application qui peut exécuter plusieurs jeux, vous pouvez rechercher tous les rapports d'exception liés à un jeu particulier. Vous pouvez ajouter autant d'éléments que vous le souhaitez à chaque dictionnaire.

## <a name="browser-exceptions"></a>Exceptions du navigateur
La plupart des exceptions de navigateur sont signalées.

Si votre page web inclut des fichiers de script à partir de réseaux de distribution de contenu ou d’autres domaines, vérifiez que votre balise de script possède l’attribut ```crossorigin="anonymous"```et que le serveur envoie des [en-têtes CORS](https://enable-cors.org/). Cela vous permettra d'obtenir une arborescence des appels de procédure et les détails des exceptions JavaScript non gérées à partir de ces ressources.

## <a name="reuse-your-telemetry-client"></a>Réutiliser votre client de télémétrie

> [!NOTE]
> TelemetryClient est destiné à être instancié une seule fois et réutilisé tout au long de la durée de vie d’une application.

Voici un exemple d’utilisation correcte de TelemetryClient.

```csharp
public class GoodController : ApiController
{
    // OK
    private static readonly TelemetryClient telemetryClient;

    static GoodController()
    {
        telemetryClient = new TelemetryClient();
    }
}
```


## <a name="web-forms"></a>Formulaires web
Pour les formulaires web, le module HTTP pourra collecter les exceptions si aucune redirection n’est configurée avec CustomErrors.

Mais si vous avez des redirections actives, ajoutez les lignes suivantes à la fonction Application_Error dans Global.asax.cs. (Ajouter un fichier Global.asax si vous n'en avez pas déjà).

```csharp
    void Application_Error(object sender, EventArgs e)
    {
      if (HttpContext.Current.IsCustomErrorEnabled && Server.GetLastError () != null)
      {
         var ai = new TelemetryClient(); // or re-use an existing instance

         ai.TrackException(Server.GetLastError());
      }
    }
```
## <a name="mvc"></a>MVC
À partir de la version 2.6 (bêta 3 et versions ultérieures) du Kit de développement logiciel (SDK) web d’Application Insights, Application Insights collecte automatiquement les exceptions non prises en charge qui sont levées dans les méthodes des contrôleurs MVC 5+. Si vous avez déjà ajouté un gestionnaire personnalisé pour suivre ces exceptions (comme dans les exemples suivants), vous pouvez le supprimer afin d’éviter un double suivi.

Il existe un certain nombre de cas que les filtres d'exception ne peuvent pas gérer. Par exemple :

* Les exceptions lancées à partir des constructeurs de contrôleur.
* Les exceptions lancées à partir des gestionnaires de messages.
* Les exceptions lancées pendant le routage.
* Les exceptions lancées pendant la sérialisation du contenu de réponse.
* Exception levée lors du démarrage de l’application.
* Exception levée dans les tâches en arrière-plan.

Toutes les exceptions *prises en charge* par l’application doivent toujours faire l’objet d’un suivi manuel.
Les exceptions non prises en charge provenant de contrôleurs aboutissent généralement à une réponse « Erreur interne du serveur » 500. Si cette réponse est construite manuellement à la suite d’une exception prise en charge (ou d’aucune exception), elle est suivie dans la télémétrie de la demande correspondante avec le `ResultCode` 500, mais le Kit SDK Application Insights n’est pas en mesure de suivre l’exception associée.

### <a name="prior-versions-support"></a>Prise en charge des versions antérieures
Si vous utilisez MVC 4 (ou une version antérieure) du Kit SDK web 2.5 d’Application Insights (ou une version antérieure), référez-vous aux exemples suivants pour effectuer le suivi des exceptions.

Si la configuration de [CustomErrors](/previous-versions/dotnet/netframework-4.0/h0hfz6fc(v=vs.100)) est `Off`, les exceptions seront alors disponibles pour être collectées par le [module HTTP](/previous-versions/dotnet/netframework-3.0/ms178468(v=vs.85)). Toutefois, si elle est `RemoteOnly` (valeur par défaut), ou `On`, l'exception ne sera alors pas disponible pour être collectée automatiquement par Application Insights. Vous pouvez corriger ce phénomène en remplaçant la [classe System.Web.Mvc.HandleErrorAttribute](/dotnet/api/system.web.mvc.handleerrorattribute?view=aspnet-mvc-5.2) et en appliquant la classe remplacée comme indiqué pour les différentes versions MVC ci-dessous ([source GitHub](https://github.com/AppInsightsSamples/Mvc2UnhandledExceptions/blob/master/MVC2App/Controllers/AiHandleErrorAttribute.cs)) :

```csharp
    using System;
    using System.Web.Mvc;
    using Microsoft.ApplicationInsights;

    namespace MVC2App.Controllers
    {
      [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = true, AllowMultiple = true)]
      public class AiHandleErrorAttribute : HandleErrorAttribute
      {
        public override void OnException(ExceptionContext filterContext)
        {
            if (filterContext != null && filterContext.HttpContext != null && filterContext.Exception != null)
            {
                //If customError is Off, then AI HTTPModule will report the exception
                if (filterContext.HttpContext.IsCustomErrorEnabled)
                {   //or reuse instance (recommended!). see note above
                    var ai = new TelemetryClient();
                    ai.TrackException(filterContext.Exception);
                }
            }
            base.OnException(filterContext);
        }
      }
    }
```

#### <a name="mvc-2"></a>MVC 2
Remplacez l'attribut HandleError par votre nouvel attribut dans vos contrôleurs.

```csharp
    namespace MVC2App.Controllers
    {
        [AiHandleError]
        public class HomeController : Controller
        {
    ...
```

[Exemple](https://github.com/AppInsightsSamples/Mvc2UnhandledExceptions)

#### <a name="mvc-3"></a>MVC 3
Enregistrez `AiHandleErrorAttribute` en tant que filtre global dans Global.asax.cs :

```csharp
    public class MyMvcApplication : System.Web.HttpApplication
    {
      public static void RegisterGlobalFilters(GlobalFilterCollection filters)
      {
         filters.Add(new AiHandleErrorAttribute());
      }
     ...
```

[Exemple](https://github.com/AppInsightsSamples/Mvc3UnhandledExceptionTelemetry)

#### <a name="mvc-4-mvc5"></a>MVC 4, MVC 5
Enregistrez AiHandleErrorAttribute en tant que filtre global dans FilterConfig.cs :

```csharp
    public class FilterConfig
    {
      public static void RegisterGlobalFilters(GlobalFilterCollection filters)
      {
        // Default replaced with the override to track unhandled exceptions
        filters.Add(new AiHandleErrorAttribute());
      }
    }
```

[Exemple](https://github.com/AppInsightsSamples/Mvc5UnhandledExceptionTelemetry)

## <a name="web-api"></a>API Web
À partir de la version 2.6 (bêta 3 et versions ultérieures) du Kit de développement logiciel (SDK) web d’Application Insights, Application Insights collecte automatiquement les exceptions non prises en charge qui sont levées dans les méthodes des contrôleurs pour WebAPI 2+. Si vous avez déjà ajouté un gestionnaire personnalisé pour suivre ces exceptions (comme dans les exemples suivants), vous pouvez le supprimer afin d’éviter un double suivi.

Il existe un certain nombre de cas que les filtres d'exception ne peuvent pas gérer. Par exemple :

* Les exceptions lancées à partir des constructeurs de contrôleur.
* Les exceptions lancées à partir des gestionnaires de messages.
* Les exceptions lancées pendant le routage.
* Les exceptions lancées pendant la sérialisation du contenu de réponse.
* Exception levée lors du démarrage de l’application.
* Exception levée dans les tâches en arrière-plan.

Toutes les exceptions *prises en charge* par l’application doivent toujours faire l’objet d’un suivi manuel.
Les exceptions non prises en charge provenant de contrôleurs aboutissent généralement à une réponse « Erreur interne du serveur » 500. Si cette réponse est construite manuellement à la suite d’une exception prise en charge (ou d’aucune exception), elle est suivie dans la télémétrie d’une demande correspondante avec le `ResultCode` 500, mais le Kit SDK Application Insights n’est pas en mesure de suivre l’exception associée.

### <a name="prior-versions-support"></a>Prise en charge des versions antérieures
Si vous utilisez WebAPI 1 (ou une version antérieure) du Kit SDK web 2.5 d’Application Insights (ou une version antérieure), référez-vous aux exemples suivants pour effectuer le suivi des exceptions.

#### <a name="web-api-1x"></a>API Web 1.x
Remplacez System.Web.Http.Filters.ExceptionFilterAttribute :

```csharp
    using System.Web.Http.Filters;
    using Microsoft.ApplicationInsights;

    namespace WebAPI.App_Start
    {
      public class AiExceptionFilterAttribute : ExceptionFilterAttribute
      {
        public override void OnException(HttpActionExecutedContext actionExecutedContext)
        {
            if (actionExecutedContext != null && actionExecutedContext.Exception != null)
            {  //or reuse instance (recommended!). see note above
                var ai = new TelemetryClient();
                ai.TrackException(actionExecutedContext.Exception);
            }
            base.OnException(actionExecutedContext);
        }
      }
    }
```

Vous pouvez ajouter cet attribut remplacé à des contrôleurs spécifiques ou l’ajouter à la configuration du filtre global dans la classe WebApiConfig :

```csharp
    using System.Web.Http;
    using WebApi1.x.App_Start;

    namespace WebApi1.x
    {
      public static class WebApiConfig
      {
        public static void Register(HttpConfiguration config)
        {
            config.Routes.MapHttpRoute(name: "DefaultApi", routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional });
            ...
            config.EnableSystemDiagnosticsTracing();

            // Capture exceptions for Application Insights:
            config.Filters.Add(new AiExceptionFilterAttribute());
        }
      }
    }
```

[Exemple](https://github.com/AppInsightsSamples/WebApi_1.x_UnhandledExceptions)

#### <a name="web-api-2x"></a>API Web 2.x
Ajoutez d'une implémentation de IExceptionLogger :

```csharp
    using System.Web.Http.ExceptionHandling;
    using Microsoft.ApplicationInsights;

    namespace ProductsAppPureWebAPI.App_Start
    {
      public class AiExceptionLogger : ExceptionLogger
      {
        public override void Log(ExceptionLoggerContext context)
        {
            if (context !=null && context.Exception != null)
            {//or reuse instance (recommended!). see note above
                var ai = new TelemetryClient();
                ai.TrackException(context.Exception);
            }
            base.Log(context);
        }
      }
    }
```

Ajoutez cela aux services dans WebApiConfig :

```csharp
    using System.Web.Http;
    using System.Web.Http.ExceptionHandling;
    using ProductsAppPureWebAPI.App_Start;

    namespace WebApi2WithMVC
    {
      public static class WebApiConfig
      {
        public static void Register(HttpConfiguration config)
        {
            // Web API configuration and services

            // Web API routes
            config.MapHttpAttributeRoutes();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
            config.Services.Add(typeof(IExceptionLogger), new AiExceptionLogger());
        }
      }
     }
```

[Exemple](https://github.com/AppInsightsSamples/WebApi_2.x_UnhandledExceptions)

Alternativement, vous pouvez :

1. Remplacer le seul gestionnaire d’exceptions avec une implémentation personnalisée de IExceptionHandler. Celle-ci est appelée uniquement lorsque l'infrastructure est toujours en mesure de choisir le message de réponse à envoyer (mais pas lorsque la connexion est abandonnée par exemple)
2. Les filtres d'exception (comme décrit dans la section sur les contrôleurs API Web 1.x ci-dessus) ne sont pas appelés dans tous les cas.

## <a name="wcf"></a>WCF
Ajoutez une classe qui étend l'attribut et implémente IErrorHandler et IServiceBehavior.

```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.ServiceModel.Description;
    using System.ServiceModel.Dispatcher;
    using System.Web;
    using Microsoft.ApplicationInsights;

    namespace WcfService4.ErrorHandling
    {
      public class AiLogExceptionAttribute : Attribute, IErrorHandler, IServiceBehavior
      {
        public void AddBindingParameters(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase,
            System.Collections.ObjectModel.Collection<ServiceEndpoint> endpoints,
            System.ServiceModel.Channels.BindingParameterCollection bindingParameters)
        {
        }

        public void ApplyDispatchBehavior(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase)
        {
            foreach (ChannelDispatcher disp in serviceHostBase.ChannelDispatchers)
            {
                disp.ErrorHandlers.Add(this);
            }
        }

        public void Validate(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase)
        {
        }

        bool IErrorHandler.HandleError(Exception error)
        {//or reuse instance (recommended!). see note above
            var ai = new TelemetryClient();

            ai.TrackException(error);
            return false;
        }

        void IErrorHandler.ProvideFault(Exception error,
            System.ServiceModel.Channels.MessageVersion version,
            ref System.ServiceModel.Channels.Message fault)
        {
        }
      }
    }

Add the attribute to the service implementations:

    namespace WcfService4
    {
        [AiLogException]
        public class Service1 : IService1
        {
         ...
```

[Exemple](https://github.com/AppInsightsSamples/WCFUnhandledExceptions)

## <a name="exception-performance-counters"></a>Compteurs de performance des exceptions
Si vous avez [installé l’agent Application Insights](./monitor-performance-live-website-now.md) sur votre serveur, vous pouvez obtenir un graphique du taux d’exceptions, mesuré par .NET. Celui-ci comprend les exceptions .NET gérées et non gérées.

Ouvrez un onglet d’explorateur de mesures, ajoutez un nouveau graphique, puis sélectionnez **Taux d’exception**sous Compteurs de performances.

.NET Framework calcule le taux en comptant le nombre d’exceptions sur un intervalle et en divisant ce nombre par la longueur de l’intervalle.

Ce chiffre sera différent du nombre d’« exceptions » calculé par le portail Application Insights, qui est basé sur les rapports TrackException. Les intervalles d’échantillonnage sont différents et le Kit de développement logiciel (SDK) n’envoie pas de rapports TrackException pour toutes les exceptions gérées et non gérées.

## <a name="next-steps"></a>Étapes suivantes
* [Surveiller REST, SQL et les autres appels aux dépendances](./asp-net-dependencies.md)
* [Surveiller les durées de chargement des pages, les exceptions du navigateur et les appels AJAX](./javascript.md)
* [Surveiller les compteurs de performances](./performance-counters.md)

