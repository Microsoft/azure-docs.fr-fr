---
title: Diagnostics et traçage de bout en bout d’Azure Service Bus | Microsoft Docs
description: Vue d’ensemble des diagnostics et du traçage de bout en bout du client Service Bus (client via tous les services impliqués dans le traitement).
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: bc7dab21fc01b624e8ab122fe883be89ea8633f6
ms.sourcegitcommit: 8be279f92d5c07a37adfe766dc40648c673d8aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97832690"
---
# <a name="distributed-tracing-and-correlation-through-service-bus-messaging"></a>Suivi distribué et corrélation via la messagerie Service Bus

L’un des défis que pose couramment le développement de microservices est la capacité à tracer l’opération d’un client à travers tous les services impliqués dans le traitement. Le traçage facilite le débogage, l’analyse des performances, les tests A/B et d’autres diagnostics standard.
L’un des aspects du problème a trait au suivi d’éléments de travail logiques. Ceux-ci comptent notamment le résultat et la latence du traitement des messages, et les appels de dépendances externes. Un autre aspect du problème est la mise en corrélation de ces événements de diagnostic au-delà des limites de traitement.

Quand un producteur envoie un message par le biais d’une file d’attente, cela se produit généralement dans l’étendue d’une autre opération logique lancée par un autre client ou service. Le consommateur continue la même opération quand il reçoit un message. Le producteur et le consommateur (et les autres services qui traitent l’opération) émettent probablement des événements de télémétrie pour tracer le flux et le résultat de l’opération. Pour mettre en corrélation de tels événements et tracer l’opération de bout en bout, chaque service qui signale des données de télémétrie doit horodater les événements avec un contexte de trace.

La messagerie Microsoft Azure Service Bus définit des propriétés de charge utile que les producteurs et les consommateurs doivent utiliser pour passer ce contexte de trace.
Le protocole est basé sur le [protocole de corrélation HTTP](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md).

| Nom de la propriété        | Description                                                 |
|----------------------|-------------------------------------------------------------|
|  Diagnostic-Id       | Identificateur unique d’un appel externe du producteur à la file d’attente. Pour explorer la logique, les considérations et le format, consultez [Request-Id dans le protocole HTTP](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md#request-id). |
|  Correlation-Context | Contexte d’opération propagé à travers tous les services impliqués dans le traitement de l’opération. Pour plus d’informations, consultez [Correlation-Context dans le protocole HTTP](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md#correlation-context). |

## <a name="service-bus-net-client-autotracing"></a>Traçage automatique du client .NET Service Bus

À compter de la version 3.0.0, [Microsoft Azure Service Bus Client pour .NET](/dotnet/api/microsoft.azure.servicebus.queueclient) fournit des points d’instrumentation de traçage qui peuvent être interceptés par des systèmes de traçage ou des parties de code client.
L’instrumentation permet le suivi de tous les appels au service de messagerie Service Bus du côté client. Si le traitement des messages est effectué avec le [modèle du gestionnaire de messages](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler), le traitement des messages est également instrumenté.

### <a name="tracking-with-azure-application-insights"></a>Suivi avec Azure Application Insights

[Microsoft Application Insights](https://azure.microsoft.com/services/application-insights/) fournit des fonctionnalités riches de monitoring des performances, notamment le suivi automagique des requêtes et des dépendances.

En fonction de votre type de projet, installez le SDK Application Insights :
- [ASP.NET](../azure-monitor/app/asp-net.md) : installez la version 2.5 (bêta 2) ou ultérieure.
- [ASP.NET Core](../azure-monitor/app/asp-net-core.md) : installez la version 2.2.0 (bêta 2) ou ultérieure.
Ces liens fournissent des détails sur l’installation du SDK, la création de ressources et la configuration du SDK (le cas échéant). Pour les applications non-ASP.NET, consultez l’article [Azure Application Insights pour les applications console](../azure-monitor/app/console.md).

Si vous utilisez le [modèle du gestionnaire de messages](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler) pour traiter les messages, votre travail est terminé : tous les appels Service Bus effectués par votre service sont automatiquement suivis et mis en corrélation avec d’autres éléments de télémétrie. Sinon, passez en revue l’exemple suivant qui illustre le suivi manuel du traitement des messages.

#### <a name="trace-message-processing"></a>Tracer le traitement des messages

```csharp
private readonly TelemetryClient telemetryClient;

async Task ProcessAsync(Message message)
{
    var activity = message.ExtractActivity();
    
    // If you are using Microsoft.ApplicationInsights package version 2.6-beta or higher, you should call StartOperation<RequestTelemetry>(activity) instead
    using (var operation = telemetryClient.StartOperation<RequestTelemetry>("Process", activity.RootId, activity.ParentId))
    {
        telemetryClient.TrackTrace("Received message");
        try 
        {
           // process message
        }
        catch (Exception ex)
        {
             telemetryClient.TrackException(ex);
             operation.Telemetry.Success = false;
             throw;
        }

        telemetryClient.TrackTrace("Done");
   }
}
```

Dans cet exemple, `RequestTelemetry` est signalé pour chaque message traité, avec un horodateur, une durée et un résultat (réussite). Les données de télémétrie comprennent également un jeu de propriétés de corrélation.
Les traces et exceptions imbriquées qui sont signalées durant le traitement des messages sont également horodatées avec des propriétés de corrélation qui les représentent comme des « enfants » de `RequestTelemetry`.

Si vous appelez des composants externes pris en charge durant le traitement des messages, ils sont également suivis et mis en corrélation automatiquement. Pour effectuer manuellement le suivi et la mise en corrélation, consultez [Suivi des opérations personnalisées avec le kit SDK .NET d’Application Insights](../azure-monitor/app/custom-operations-tracking.md).

Si vous exécutez un code externe en plus du SDK Application Insights, attendez-vous à constater une **durée** plus longue lors de l’affichage des journaux Application Insights. 

![Durée plus longue dans le journal Application Insights](./media/service-bus-end-to-end-tracing/longer-duration.png)

Cela ne signifie pas qu’il y a eu un retard lors de la réception du message. Dans ce scénario, le message a déjà été reçu, car il est passé en tant que paramètre au code du SDK. De plus, la balise **name** dans les journaux App Insights (**Process**) indique que le message est en cours de traitement par votre code de traitement d’événement externe. Ce problème n’est pas lié à Azure. Au lieu de cela, ces métriques font référence à l’efficacité de votre code externe, étant donné que le message a déjà été reçu de Service Bus. Consultez [ce fichier sur GitHub](https://github.com/Azure/azure-sdk-for-net/blob/4bab05144ce647cc9e704d46d3763de5f9681ee0/sdk/servicebus/Microsoft.Azure.ServiceBus/src/ServiceBusDiagnosticsSource.cs) pour voir où la balise **Process** est générée et assignée une fois que le message a été reçu de Service Bus. 

### <a name="tracking-without-tracing-system"></a>Suivi sans un système de traçage
Si votre système de traçage ne prend pas en charge le suivi automatique des appels Service Bus, songez à l’ajouter dans un système de traçage ou dans votre application. Cette section décrit les événements de diagnostic envoyés par Service Bus .NET Client.  

Service Bus .NET Client est instrumenté à l’aide des primitives de traçage .NET [System.Diagnostics.Activity](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) et [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md).

`Activity` sert de contexte de trace, tandis que `DiagnosticSource` est un mécanisme de notification. 

S’il n’y a aucun écouteur pour les événements DiagnosticSource, l’instrumentation est désactivée et ne génère pas de frais. DiagnosticSource donne tout le contrôle à l’écouteur :
- L’écouteur contrôle les sources et les événements qu’il écoute
- L’écouteur contrôle l’échantillonnage et la fréquence des événements
- Les événements sont envoyés avec une charge utile qui fournit un contexte complet, ce qui vous permet d’accéder à l’objet Message durant l’événement et de le modifier

Avant de passer à l’implémentation, passez en revue le [guide de l’utilisateur de DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md).

Nous allons créer un écouteur pour les événements Service Bus dans une application ASP.NET Core qui écrit des journaux d’activité avec Microsoft.Extension.Logger.
L’abonnement à DiagnosticSource s’effectue à l’aide de la bibliothèque [System.Reactive.Core](https://www.nuget.org/packages/System.Reactive.Core) (vous pouvez aussi vous abonner facilement à DiagnosticSource sans cette bibliothèque).

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory factory, IApplicationLifetime applicationLifetime)
{
    // configuration...

    var serviceBusLogger = factory.CreateLogger("Microsoft.Azure.ServiceBus");

    IDisposable innerSubscription = null;
    IDisposable outerSubscription = DiagnosticListener.AllListeners.Subscribe(delegate (DiagnosticListener listener)
    {
        // subscribe to the Service Bus DiagnosticSource
        if (listener.Name == "Microsoft.Azure.ServiceBus")
        {
            // receive event from Service Bus DiagnosticSource
            innerSubscription = listener.Subscribe(delegate (KeyValuePair<string, object> evnt)
            {
                // Log operation details once it's done
                if (evnt.Key.EndsWith("Stop"))
                {
                    Activity currentActivity = Activity.Current;
                    TaskStatus status = (TaskStatus)evnt.Value.GetProperty("Status");
                    serviceBusLogger.LogInformation($"Operation {currentActivity.OperationName} is finished, Duration={currentActivity.Duration}, Status={status}, Id={currentActivity.Id}, StartTime={currentActivity.StartTimeUtc}");
                }
            });
        }
    });

    applicationLifetime.ApplicationStopping.Register(() =>
    {
        outerSubscription?.Dispose();
        innerSubscription?.Dispose();
    });
}
```

Dans cet exemple, l’écouteur journalise la durée, le résultat, l’identificateur unique et l’heure de début de chaque opération Service Bus.

#### <a name="events"></a>Événements

Pour chaque opération, deux événements sont envoyés : « Start » et « Stop ». Dans la plupart des cas, seuls les événements « Stop » présentent un intérêt. Ceux-ci fournissent le résultat de l’opération, l’heure de début et la durée sous la forme de propriétés Activity.

La charge utile d’événement, qui fournit un écouteur avec le contexte de l’opération, réplique les paramètres entrants et la valeur de retour de l’API. La charge utile d’événement « Stop » contient toutes les propriétés de la charge utile d’événement « Start ». Vous pouvez donc ignorer l’événement « Start ».

Tous les événements ont également des propriétés « Entity » et « Endpoint » qui sont omises dans le tableau ci-dessous.
  * `string Entity` : Nom de l’entité (file d’attente, rubrique, etc.)
  * `Uri Endpoint` : URL du point de terminaison Service Bus

Chaque événement « Stop » comprend la propriété `Status` qui indique le `TaskStatus` à la fin de l’opération asynchrone. Par souci de simplicité, ces éléments ne figurent pas non plus dans le tableau suivant.

Voici la liste complète des opérations instrumentées :

| Nom d’opération | API suivie | Propriétés de charge utile spécifiques|
|----------------|-------------|---------|
| Microsoft.Azure.ServiceBus.Send | [MessageSender.SendAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.sendasync) | `IList<Message> Messages` - Liste des messages en cours d’envoi |
| Microsoft.Azure.ServiceBus.ScheduleMessage | [MessageSender.ScheduleMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.schedulemessageasync) | `Message Message` - Message en cours de traitement<br/>`DateTimeOffset ScheduleEnqueueTimeUtc` - Décalage de message planifié<br/>`long SequenceNumber` - Numéro de séquence du message planifié (charge utile d’événement « Stop ») |
| Microsoft.Azure.ServiceBus.Cancel | [MessageSender.CancelScheduledMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.cancelscheduledmessageasync) | `long SequenceNumber` - Numéro de séquence du message à annuler | 
| Microsoft.Azure.ServiceBus.Receive | [MessageReceiver.ReceiveAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.receiveasync) | `int RequestedMessageCount` - Nombre maximal de messages pouvant être reçus.<br/>`IList<Message> Messages` - Liste des messages reçus (charge utile d’événement « Stop ») |
| Microsoft.Azure.ServiceBus.Peek | [MessageReceiver.PeekAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.peekasync) | `int FromSequenceNumber` - Point de départ à partir duquel parcourir un lot de messages.<br/>`int RequestedMessageCount` - Nombre de messages à récupérer.<br/>`IList<Message> Messages` - Liste des messages reçus (charge utile d’événement « Stop ») |
| Microsoft.Azure.ServiceBus.ReceiveDeferred | [MessageReceiver.ReceiveDeferredMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.receivedeferredmessageasync) | `IEnumerable<long> SequenceNumbers` - Liste contenant les numéros de séquence à recevoir.<br/>`IList<Message> Messages` - Liste des messages reçus (charge utile d’événement « Stop ») |
| Microsoft.Azure.ServiceBus.Complete | [MessageReceiver.CompleteAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.completeasync) | `IList<string> LockTokens` - Liste contenant les jetons de verrouillage des messages correspondants à terminer.|
| Microsoft.Azure.ServiceBus.Abandon | [MessageReceiver.AbandonAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.abandonasync) | `string LockToken` - Jeton de verrouillage du message correspondant à abandonner. |
| Microsoft.Azure.ServiceBus.Defer | [MessageReceiver.DeferAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deferasync) | `string LockToken` - Jeton de verrouillage du message correspondant à différer. | 
| Microsoft.Azure.ServiceBus.DeadLetter | [MessageReceiver.DeadLetterAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deadletterasync) | `string LockToken` - Jeton de verrouillage du message correspondant à placer dans la file d’attente de lettres mortes. | 
| Microsoft.Azure.ServiceBus.RenewLock | [MessageReceiver.RenewLockAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync) | `string LockToken` - Jeton de verrouillage du message correspondant dont le verrouillage doit être renouvelé.<br/>`DateTime LockedUntilUtc` - Date et heure d’expiration du nouveau jeton de verrouillage au format UTC. (Charge utile d’événement « Stop »)|
| Microsoft.Azure.ServiceBus.Process | Fonction lambda du gestionnaire de messages fournie dans [IReceiverClient.RegisterMessageHandler](/dotnet/api/microsoft.azure.servicebus.core.ireceiverclient.registermessagehandler) | `Message Message` - Message en cours de traitement. |
| Microsoft.Azure.ServiceBus.ProcessSession | Fonction lambda du gestionnaire de sessions de message fournie dans [IQueueClient.RegisterSessionHandler](/dotnet/api/microsoft.azure.servicebus.iqueueclient.registersessionhandler) | `Message Message` - Message en cours de traitement.<br/>`IMessageSession Session` - Session en cours de traitement |
| Microsoft.Azure.ServiceBus.AddRule | [SubscriptionClient.AddRuleAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.addruleasync) | `RuleDescription Rule` - Description de règle qui fournit la règle à ajouter. |
| Microsoft.Azure.ServiceBus.RemoveRule | [SubscriptionClient.RemoveRuleAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.removeruleasync) | `string RuleName` - Nom de la règle à supprimer. |
| Microsoft.Azure.ServiceBus.GetRules | [SubscriptionClient.GetRulesAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.getrulesasync) | `IEnumerable<RuleDescription> Rules` - Toutes les règles associées à l’abonnement. (Charge utile « Stop » uniquement) |
| Microsoft.Azure.ServiceBus.AcceptMessageSession | [ISessionClient.AcceptMessageSessionAsync](/dotnet/api/microsoft.azure.servicebus.isessionclient.acceptmessagesessionasync) | `string SessionId` - ID de session présent dans les messages. |
| Microsoft.Azure.ServiceBus.GetSessionState | [IMessageSession.GetStateAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.getstateasync) | `string SessionId` - ID de session présent dans les messages.<br/>`byte [] State` - État de session (charge utile d’événement « Stop ») |
| Microsoft.Azure.ServiceBus.SetSessionState | [IMessageSession.SetStateAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.setstateasync) | `string SessionId` - ID de session présent dans les messages.<br/>`byte [] State` - État de la session |
| Microsoft.Azure.ServiceBus.RenewSessionLock | [IMessageSession.RenewSessionLockAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.renewsessionlockasync) | `string SessionId` - ID de session présent dans les messages. |
| Microsoft.Azure.ServiceBus.Exception | Toute API instrumentée| `Exception Exception` - Instance d’exception |

Dans chaque événement, vous pouvez accéder à `Activity.Current` qui contient le contexte de l’opération actuelle.

#### <a name="logging-additional-properties"></a>Journalisation de propriétés supplémentaires

`Activity.Current` fournit un contexte détaillé de l’opération actuelle et de ses parents. Pour plus d’informations, consultez la [documentation Activity](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md).
L’instrumentation Service Bus fournit des informations supplémentaires dans les `Activity.Current.Tags` qui, le cas échéant, contiennent `MessageId` et `SessionId`.

Les activités qui font le suivi des événements « Receive », «Peek » et « ReceiveDeferred » peuvent également avoir la balise `RelatedTo`. Celle-ci contient une liste distincte des `Diagnostic-Id` des messages reçus à cette occasion.
Cette opération peut entraîner la réception de plusieurs messages non liés. Par ailleurs, `Diagnostic-Id` n’étant pas connu au démarrage de l’opération, les opérations « Receive » peuvent uniquement être mises en corrélation avec les opérations « Process » au moyen de cette balise. C’est utile lors de l’analyse des problèmes de performances pour déterminer le temps qu’il a fallu pour recevoir le message.

Un moyen efficace de journaliser les balises consiste à les itérer. Voici à quoi ressemble l’exemple précédent après l’ajout de balises : 

```csharp
Activity currentActivity = Activity.Current;
TaskStatus status = (TaskStatus)evnt.Value.GetProperty("Status");

var tagsList = new StringBuilder();
foreach (var tags in currentActivity.Tags)
{
    tagsList.Append($", {tags.Key}={tags.Value}");
}

serviceBusLogger.LogInformation($"{currentActivity.OperationName} is finished, Duration={currentActivity.Duration}, Status={status}, Id={currentActivity.Id}, StartTime={currentActivity.StartTimeUtc}{tagsList}");
```

#### <a name="filtering-and-sampling"></a>Filtrage et échantillonnage

Dans certains cas, il est souhaitable de journaliser uniquement une partie des événements pour réduire la surcharge des performances ou la consommation du stockage. Vous pouvez journaliser uniquement les événements « Stop » (comme dans l’exemple précédent) ou échantillonner un pourcentage des événements. 
`DiagnosticSource` offre un moyen d’y parvenir avec le prédicat `IsEnabled`. Pour plus d’informations, consultez [Filtrage basé sur le contexte dans DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md#context-based-filtering).

`IsEnabled` peut être appelé plusieurs fois pour une opération unique afin de minimiser l’impact sur les performances.

`IsEnabled` est appelé dans la séquence suivante :

1. `IsEnabled(<OperationName>, string entity, null)`, par exemple `IsEnabled("Microsoft.Azure.ServiceBus.Send", "MyQueue1")`. Notez l’absence de « Start » ou de « Stop » à la fin. Utilisez-le pour exclure des opérations ou des files d’attente particulières. Si le rappel retourne `false`, les événements de l’opération ne sont pas envoyés.

   * Pour les opérations « Process » et « ProcessSession », vous recevez également le rappel `IsEnabled(<OperationName>, string entity, Activity activity)`. Utilisez-le pour filtrer des événements en fonction des propriétés des balises ou de `activity.Id`.
  
2. `IsEnabled(<OperationName>.Start)`, par exemple `IsEnabled("Microsoft.Azure.ServiceBus.Send.Start")`. Vérifie si l’événement « Start » doit être déclenché. Le résultat affecte uniquement les événements « Start », mais les opérations supplémentaires d’instrumentation n’en dépendent pas.

`IsEnabled` n’est pas disponible pour l’événement « Stop ».

Si le résultat d’une opération est une exception, `IsEnabled("Microsoft.Azure.ServiceBus.Exception")` est appelé. Vous pouvez uniquement vous abonner aux événements « Exception » et empêcher le reste de l’instrumentation. Dans ce cas, vous devez encore gérer ces exceptions. Toute autre instrumentation étant désactivée, ne vous attendez pas à ce que le contexte de trace soit passé avec les messages du consommateur au producteur.

Vous pouvez également utiliser `IsEnabled` pour implémenter des stratégies d’échantillonnage. L’échantillonnage basé sur `Activity.Id` ou `Activity.RootId` garantit des résultats cohérents (à condition qu’il soit propagé par le système de suivi ou par votre propre code).

En présence de plusieurs écouteurs `DiagnosticSource` pour la même source, il suffit qu’un seul écouteur accepte l’événement. L’appel de `IsEnabled` n’est donc pas garanti.

## <a name="next-steps"></a>Étapes suivantes

* [Corrélation dans Application Insights](../azure-monitor/app/correlation.md)
* [Monitoring des dépendances Application Insights](../azure-monitor/app/asp-net-dependencies.md) pour déterminer si REST, SQL ou d’autres ressources externes ralentissent vos opérations.
* [Suivi des opérations personnalisées avec le kit SDK .NET d’Application Insights](../azure-monitor/app/custom-operations-tracking.md)
