---
title: Créer des événements de journal à partir d’une application .NET
description: Apprenez comment ajouter la journalisation à votre application .NET Service Fabric hébergé sur un cluster Azure ou un cluster autonome.
author: srrengar
ms.topic: conceptual
ms.date: 03/27/2018
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: a36425acf42a469c7f48b2e954bdacfdfcce1b10
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89011478"
---
# <a name="add-logging-to-your-service-fabric-application"></a>Ajouter la journalisation à votre application Service Fabric

Votre application doit fournir suffisamment d’informations pour permettre un débogage approfondi en cas de problème. La journalisation est l’un des éléments les plus importants que vous puissiez ajouter à votre application Service Fabric. Lorsqu’une défaillance se produit, une journalisation efficace vous offre des options d’analyse du dysfonctionnement. En analysant les modèles du journal, vous pouvez identifier des moyens d’améliorer les performances ou la conception de votre application. Ce document évoque différentes options de journalisation.

## <a name="eventflow"></a>EventFlow

La suite de la [bibliothèque EventFlow](https://github.com/Azure/diagnostics-eventflow) donne les moyens aux applications de définir les données de diagnostic à collecter, ainsi que l’emplacement dans lesquelles les générer. Les données de diagnostic peuvent être de différentes natures, de compteurs de performances à des traces d’application. La solution s’exécute dans le même processus que l’application, la surcharge de communication est ainsi réduite ici. Pour plus d’informations sur EventFlow et Service Fabric, consultez la section [Agrégation d’événements Azure Service Fabric avec EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md).

### <a name="using-structured-eventsource-events"></a>Utilisation d’événements EventSource structurés

La définition des événements de message par cas d’utilisation vous permet d’empaqueter les données sur l’événement, dans le contexte de l’événement. Vous pouvez plus facilement effectuer des recherches et filtrer en fonction des noms ou des valeurs des propriétés spécifiques de l’événement. La structuration de la sortie d’instrumentation facilite la lecture, mais elle nécessite plus de réflexion et de temps pour la définition d’un événement à chaque cas d’utilisation. 

Certaines définitions d’événement peuvent être partagées dans toute l’application. Par exemple, un événement de commencement ou d’arrêt de méthode peut être réutilisé dans plusieurs services d’une même application. Un service spécifique d’un domaine, comme un système de commande, peut est associé à un événement **CreateOrder**, qui présente son propre événement unique. Cette approche est susceptible de générer de nombreux événements et peut requérir la coordination des identificateurs entre les équipes de projet. 

```csharp
[EventSource(Name = "MyCompany-VotingState-VotingStateService")]
internal sealed class ServiceEventSource : EventSource
{
    public static readonly ServiceEventSource Current = new ServiceEventSource();

    // The instance constructor is private to enforce singleton semantics.
    private ServiceEventSource() : base() { }

    ...

    // The ServiceTypeRegistered event contains a unique identifier, an event attribute that defined the event, and the code implementation of the event.
    private const int ServiceTypeRegisteredEventId = 3;
    [Event(ServiceTypeRegisteredEventId, Level = EventLevel.Informational, Message = "Service host process {0} registered service type {1}", Keywords = Keywords.ServiceInitialization)]
    public void ServiceTypeRegistered(int hostProcessId, string serviceType)
    {
        WriteEvent(ServiceTypeRegisteredEventId, hostProcessId, serviceType);
    }

    // The ServiceHostInitializationFailed event contains a unique identifier, an event attribute that defined the event, and the code implementation of the event.
    private const int ServiceHostInitializationFailedEventId = 4;
    [Event(ServiceHostInitializationFailedEventId, Level = EventLevel.Error, Message = "Service host initialization failed", Keywords = Keywords.ServiceInitialization)]
    public void ServiceHostInitializationFailed(string exception)
    {
        WriteEvent(ServiceHostInitializationFailedEventId, exception);
    }

    ...

```

### <a name="using-eventsource-generically"></a>Utilisation générale d’EventSource

Étant donné la complexité de la définition d’événements spécifiques, de nombreuses personnes définissent quelques événements avec un ensemble commun de paramètres qui sortent généralement les informations sous forme de chaîne. Une grande partie de la structure est perdue, ce qui complique les recherches et le filtrage des résultats. Dans le cadre de cette approche, quelques événements correspondant généralement aux niveaux de journalisation sont définis. L’extrait de code suivant définit un message d’erreur et de débogage :

```csharp
[EventSource(Name = "MyCompany-VotingState-VotingStateService")]
internal sealed class ServiceEventSource : EventSource
{
    public static readonly ServiceEventSource Current = new ServiceEventSource();

    // The Instance constructor is private, to enforce singleton semantics.
    private ServiceEventSource() : base() { }

    ...

    private const int DebugEventId = 10;
    [Event(DebugEventId, Level = EventLevel.Verbose, Message = "{0}")]
    public void Debug(string msg)
    {
        WriteEvent(DebugEventId, msg);
    }

    private const int ErrorEventId = 11;
    [Event(ErrorEventId, Level = EventLevel.Error, Message = "Error: {0} - {1}")]
    public void Error(string error, string msg)
    {
        WriteEvent(ErrorEventId, error, msg);
    }

    ...

```

Une approche hybride d’instrumentation structurée et générique peut aussi fonctionner. L’instrumentation structurée sert à créer des rapports sur les erreurs et les mesures. Des événements génériques peuvent être employés pour la journalisation détaillée utilisée par les ingénieurs lors de la résolution des problèmes.

## <a name="microsoftextensionslogging"></a>Microsoft.Extensions.Logging

La journalisation ASP.NET Core ([package Microsoft.Extensions.Logging NuGet](https://www.nuget.org/packages/Microsoft.Extensions.Logging)) est une infrastructure de journalisation qui fournit une API de journalisation standard pour votre application. La prise en charge pour d’autres serveurs principaux de journalisation peut être raccordée à la journalisation ASP.NET Core. Cela vous octroie un large éventail de prise en charge de la journalisation pour votre application, sans modification d’un volume important de code.

1. Ajoutez le package NuGet **Microsoft.Extensions.Logging** au projet à instrumenter. Par ailleurs, ajoutez les packages fournisseur. Pour plus d’informations, consultez l’article [Logging in ASP.NET Core](/aspnet/core/fundamentals/logging) (Journalisation dans ASP.NET Core).
2. Ajoutez une directive **using** pour **Microsoft.Extensions.Logging** à votre fichier de service.
3. Définissez une variable privée dans votre classe de service.

   ```csharp
   private ILogger _logger = null;
   ```

4. Dans le constructeur de votre classe de service, ajoutez le code suivant :

   ```csharp
   _logger = new LoggerFactory().CreateLogger<Stateless>();
   ```

5. Démarrez l’instrumentation de votre code dans vos méthodes. Voici quelques exemples :

   ```csharp
   _logger.LogDebug("Debug-level event from Microsoft.Logging");
   _logger.LogInformation("Informational-level event from Microsoft.Logging");

   // In this variant, we're adding structured properties RequestName and Duration, which have values MyRequest and the duration of the request.
   // Later in the article, we discuss why this step is useful.
   _logger.LogInformation("{RequestName} {Duration}", "MyRequest", requestDuration);
   ```

### <a name="using-other-logging-providers"></a>Utilisation d’autres fournisseurs de journalisation

Certains fournisseurs tiers utilisent l’approche décrite dans la section précédente, notamment [Serilog](https://serilog.net/), [NLog](https://nlog-project.org/) et [Loggr](https://github.com/imobile3/Loggr.Extensions.Logging). Vous pouvez connecter chacun d’eux à la journalisation ASP.NET Core ou les utiliser séparément. Serilog offre une fonctionnalité qui enrichit tous les messages envoyés par un enregistreur d’événements. Cette fonctionnalité peut être utile pour obtenir le nom, le type et les informations de partition du service. Pour utiliser cette fonctionnalité dans l’infrastructure ASP.NET Core, procédez comme suit :

1. Ajoutez les packages NuGet **Serilog**, **Serilog.Extensions.Logging**, **Serilog.Sinks.Literate** et **Serilog.Sinks.Observable** à votre projet. 
2. Créez une instance `LoggerConfiguration` et l’instance de l’enregistreur d’événements.

   ```csharp
   Log.Logger = new LoggerConfiguration().WriteTo.LiterateConsole().CreateLogger();
   ```

3. Ajoutez un argument `Serilog.ILogger` au constructeur de service et exécutez l’enregistreur d’événements nouvellement créé.

   ```csharp
   ServiceRuntime.RegisterServiceAsync("StatelessType", context => new Stateless(context, Log.Logger)).GetAwaiter().GetResult();
   ```

4. Dans le constructeur de service, crée des enrichisseurs de propriétés pour **ServiceTypeName**, **ServiceName**, **PartitionId** et **InstanceId**.

   ```csharp
   public Stateless(StatelessServiceContext context, Serilog.ILogger serilog)
       : base(context)
   {
       PropertyEnricher[] properties = new PropertyEnricher[]
       {
           new PropertyEnricher("ServiceTypeName", context.ServiceTypeName),
           new PropertyEnricher("ServiceName", context.ServiceName),
           new PropertyEnricher("PartitionId", context.PartitionId),
           new PropertyEnricher("InstanceId", context.ReplicaOrInstanceId),
       };

       serilog.ForContext(properties);

       _logger = new LoggerFactory().AddSerilog(serilog.ForContext(properties)).CreateLogger<Stateless>();
   }
   ```

5. Instrumentez le code de la même manière que si vous utilisiez ASP.NET Core sans Serilog.

   >[!NOTE]
   >Nous vous *déconseillons* d’utiliser la variable `Log.Logger` statique avec l’exemple précédent. En effet, Service Fabric peut héberger plusieurs instances du même type de service dans un seul processus. Si vous utilisez la variable `Log.Logger` statique, les valeurs du dernier enregistreur des enrichisseurs de propriétés seront affichées pour toutes les instances en cours d’exécution. C’est l’une des raisons pour lesquelles la variable _logger est une variable membre privée de la classe de service. En outre, vous devez mettre la variable `_logger` à disposition du code commun, qui est susceptible d’être utilisé pour différents services.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [la surveillance des applications dans Service Fabric](service-fabric-diagnostics-event-generation-app.md).
- En savoir plus sur la journalisation avec [EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md) et [Diagnostics Azure pour Windows](service-fabric-diagnostics-event-aggregation-wad.md).
