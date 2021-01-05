---
title: Surveiller les API avec Gestion des API Azure, Event Hubs et Moesif
titleSuffix: Azure API Management
description: Exemple d’application illustrant la stratégie log-to-eventhub par la connexion de Gestion des API Azure, d’Azure Event Hubs et de Moesif pour la journalisation HTTP et la surveillance
services: api-management
documentationcenter: ''
author: darrelmiller
manager: erikre
editor: ''
ms.assetid: c528cf6f-5f16-4a06-beea-fa1207541a47
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 01/23/2018
ms.author: apimpm
ms.openlocfilehash: abb9cbb73f8957cec2cb3240bbf186623b9b2ef9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88205500"
---
# <a name="monitor-your-apis-with-azure-api-management-event-hubs-and-moesif"></a>Monitor vos API avec Gestion des API Azure, Event Hubs et Moesif
Le [service de gestion des API](api-management-key-concepts.md) fournit de nombreuses fonctionnalités pour améliorer le traitement des requêtes HTTP envoyées à votre API HTTP. Toutefois, l’existence des demandes et réponses est temporaire. La demande est effectuée et elle transite par le service de gestion des API vers le serveur principal de votre API. Votre API traite la requête et une réponse retourne vers le consommateur d’API. Le service Gestion des API conserve certaines statistiques importantes sur les API à des fins d’affichage dans le tableau de bord du portail Azure, mais au-delà, les détails disparaissent.

En utilisant la stratégie log-to-eventhub dans le service Gestion des API, vous pouvez envoyer n’importe quel détail de la demande et la réponse à un [hub d’événements Azure](../event-hubs/event-hubs-about.md). Il existe de nombreuses raisons pour que vous vouliez générer des événements des messages HTTP et les envoyer à vos API. Certains exemples incluent une piste d’audit des mises à jour, une analyse des usages, une alerte en cas d’exception et l’intégration de tiers.

Cet article montre comment recueillir l’ensemble du message de demande et de réponse HTTP, l’envoyer à un hub d’événements, puis relayer le message vers un service tiers fournissant des services de journalisation et de surveillance HTTP.

## <a name="why-send-from-api-management-service"></a>Pourquoi effectuer l’envoi depuis un service de gestion d’API ?
Il est possible d’écrire un intergiciel (middleware) HTTP pouvant s’intégrer à des infrastructures d’API HTTP pour capturer les requêtes et réponses HTTP et les introduire dans la journalisation et la surveillance des systèmes. L’inconvénient de cette approche est que le middleware HTTP doit être intégré dans l’API de serveur principal et doit correspondre à la plate-forme de l’API. S’il existe plusieurs API, chacune d’elles doit déployer le middleware. Il existe souvent des raisons pour lesquelles les API de serveur principal ne peuvent pas être mises à jour.

L’utilisation du service de gestion des API Azure à intégrer à l’infrastructure de journalisation fournit une solution centralisée et indépendante de la plate-forme. Ce service est également évolutif, en partie en raison des fonctionnalités de [géo-réplication](api-management-howto-deploy-multi-region.md) de la gestion des API Azure.

## <a name="why-send-to-an-azure-event-hub"></a>Pourquoi envoyer à un hub d’événements Azure ?
Il est judicieux de se demander pourquoi créer une stratégie spécifique aux hubs d’événements Azure ? Il existe plusieurs endroits où je souhaite consigner mes demandes. Pourquoi ne pas simplement envoyer les demandes directement à la destination finale ?  C’est une option. Toutefois, durant les demandes de connexion depuis un service de gestion des API, il est nécessaire de prendre en compte l’impact de messages de journalisation sur les performances de l’API. L’augmentation progressive de charge peut être traitées par l’augmentation du nombre d’instances disponibles de composants système ou par le biais de la géo-réplication. Cependant, de courts pics de trafic peuvent entraîner des retards de demandes si les demandes d’infrastructures de journalisation commencent à ralentir en raison de la charge.

Les hubs d’événements Azure sont conçus pour accepter d’énormes volumes de données en entre, avec la possibilité de gérer un nombre d’événements bien plus élevé que le nombre de requêtes HTTP des processus de l’API. Le hub d’événements agit comme une sorte de tampon sophistiqué entre votre service de gestion des API et l’infrastructure qui stocke et traite les messages. Cela garantit que les performances de votre API ne seront pas diminuées à cause de l’infrastructure de journalisation.

Une fois les données transmises à un hub d’événements, elles y restent jusqu’à leur traitement par les consommateurs du hub d’événements. Le hub d’événements ne se soucie pas du mode de traitement ; il se contente de s’assurer que le message sera bien remis.

Event Hubs a la possibilité de faire circuler les événements de flux de données à plusieurs groupes de consommateurs. Ainsi, les événements doivent être traités par des systèmes différents. Cela permet la prise en charge de nombreux scénarios d’intégration sans ajouter de retard de traitement des requêtes d’API dans le service de gestion d’API, car un seul événement doit être généré.

## <a name="a-policy-to-send-applicationhttp-messages"></a>Une stratégie pour envoyer des messages d’application/http
Un hub d’événements accepte des données d’événement en tant que chaîne simple. C’est vous qui décidez du contenu de cette chaîne. Pour empaqueter une requête HTTP et l’envoyer à Event Hubs, nous devons mettre en forme la chaîne avec les informations de demande ou de réponse. Dans de telles situations, s’il existe un format que nous pouvons réutiliser, nous n’avons pas à rédiger notre propre code d’analyse. Au départ, j’ai envisagé d’utiliser le [HAR](http://www.softwareishard.com/blog/har-12-spec/) pour envoyer des requêtes et des réponses HTTP. Cependant, ce format est optimisé pour stocker une séquence de requêtes HTTP dans un format basé sur JSON. Il contenait un certain nombre d’éléments obligatoires qui ajoutait un degré de complexité inutile pour le scénario de transfert du message HTTP sur le réseau.

Une alternative consistait à utiliser le type de support `application/http` , comme décrit dans la spécification HTTP [RFC 7230](https://tools.ietf.org/html/rfc7230). Ce type de média utilise le même format que celui qui est utilisé pour envoyer des messages HTTP sur le réseau, mais l’intégralité du message peut être placée dans le corps d’une autre requête HTTP. Dans ce cas, nous allons simplement utiliser le corps comme message à envoyer à Event Hubs. Heureusement, il existe un analyseur dans les bibliothèques [Microsoft ASP.NET Web API 2.2 Client](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/) qui peut analyser ce format et le convertir en objets `HttpRequestMessage` et `HttpResponseMessage` natifs.

Pour être en mesure de créer ce message, nous devons utiliser des [expressions de stratégie](./api-management-policy-expressions.md) en langage C# dans la Gestion des API Azure. Voici la stratégie qui envoie un message de requête HTTP aux hubs d’événements Azure.

```xml
<log-to-eventhub logger-id="conferencelogger" partition-id="0">
@{
   var requestLine = string.Format("{0} {1} HTTP/1.1\r\n",
                                               context.Request.Method,
                                               context.Request.Url.Path + context.Request.Url.QueryString);

   var body = context.Request.Body?.As<string>(true);
   if (body != null && body.Length > 1024)
   {
       body = body.Substring(0, 1024);
   }

   var headers = context.Request.Headers
                          .Where(h => h.Key != "Authorization" && h.Key != "Ocp-Apim-Subscription-Key")
                          .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                          .ToArray<string>();

   var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

   return "request:"   + context.Variables["message-id"] + "\n"
                       + requestLine + headerString + "\r\n" + body;
}
</log-to-eventhub>
```

### <a name="policy-declaration"></a>Déclaration de stratégie
Il y a quelques éléments particuliers à examiner sur cette expression de stratégie. La stratégie log-to-eventhub possède un attribut appelé logger-id qui fait référence au nom d’enregistreur d’événements qui a été créé dans le service Gestion des API. Vous trouverez des informations détaillées pour configurer un enregistreur de hubs d’événements dans le service Gestion des API dans le document [Comment enregistrer des événements sur Azure Event Hubs dans Gestion des API Azure](api-management-howto-log-event-hubs.md). Le second attribut est un paramètre facultatif qui donne à Event Hubs la partition dans laquelle stocker le message. Event Hubs utilise des partitions pour activer la scalabilité et en nécessite au moins deux. La livraison ordonnée des messages est garantie uniquement au sein d’une partition. Si nous n’indiquons pas à Event Hubs la partition dans laquelle placer le message, il utilise un algorithme de répétition alternée pour répartir la charge. Cependant, certains de nos messages peuvent être traités dans le désordre

### <a name="partitions"></a>Partitions
Pour vérifier que nos messages sont remis aux consommateurs dans l’ordre et tirer parti de la fonctionnalité de distribution de charge des partitions, j’ai choisi d’envoyer des messages de demande HTTP à une partition et les messages de réponse HTTP sur une deuxième partition. Cela garantit une distribution régulière de charge et nous pouvons garantir que toutes les demandes seront consommées dans l’ordre, de même que toutes les réponses. Il est possible à une réponse d’être consommé avant la demande correspondante, mais ce n’est pas un problème, car nous avons un mécanisme différent pour mettre en corrélation des demandes et des réponses et nous savons que les demandes viennent toujours avant les réponses.

### <a name="http-payloads"></a>Charges utiles HTTP
Après la génération du `requestLine`, nous vérifions si le corps de la requête doit être tronqué. Le corps de la demande est tronqué à 1024 uniquement. Cette valeur peut être augmentée ; cependant, les messages de hub d’événements individuels étant limités à 256 Ko, il est probable que certains corps des messages HTTP ne tiennent pas dans un seul message. Lors de la journalisation et de l’analyse, une quantité significative d’informations peut être dérivée de la ligne et des en-têtes de requête HTTP. De nombreuses demandes d’API ne renvoient qu’un corps de petite taille, et donc, la perte de valeur d’informations obtenue quand on tronque les corps volumineux est minime par rapport à la réduction des coûts de transfert, de traitement et de stockage pour garder tous les contenus du corps. Dernière remarque sur le traitement du corps : nous devons transmettre `true` à la méthode `As<string>()`, car nous lisons le contenu du corps, mais souhaitons également que l’API de service principal soit en mesure de lire le corps. En mettant cette méthode sur true, nous faisons en sorte que le corps soit mis en mémoire cache et puisse être lu une seconde fois. Cela peut avoir son importance si l’API réalise le chargement de fichiers volumineux ou utilise l’interrogation longue. Dans ces cas, il est préférable d’éviter carrément la lecture du corps.

### <a name="http-headers"></a>En-têtes HTTP
Les en-têtes HTTP peuvent être transférés au format du message sous forme de paire clé/valeur simple. Nous avons choisi de supprimer certains champs de sécurité sensibles, pour éviter la fuite inutile des informations d’identification. Il est peu probable que les clés d’API et les autres informations d’identification à utiliser à des fins d’analyse. Si nous souhaitons effectuer une analyse sur l’utilisateur et le produit qu’il utilise, nous pouvons le faire à partir de l’objet `context` et l’ajouter au message.

### <a name="message-metadata"></a>Métadonnées de message
Lorsque vous créez un message complet à envoyer au hub d’événements, la première ligne ne fait par vraiment partie du message `application/http` . La première ligne est composée de métadonnées supplémentaires pour déterminer si le message est une demande ou un message de réponse et un ID de message qui est utilisé pour corréler les demandes aux réponses. L’ID de message est créé à l’aide d’une autre stratégie qui ressemble à ceci :

```xml
<set-variable name="message-id" value="@(Guid.NewGuid())" />
```

Il est possible de créer le message de demande, de le stocker dans une variable jusqu’à ce que la réponse soit renvoyée, puis d’envoyer la demande et la réponse sous forme de message unique. Toutefois, en envoyant la demande et la réponse indépendamment l’une de l’autre et en utilisant un id de message pour mettre les deux en corrélation, nous obtenons un davantage de flexibilité en matière de taille du message, la possibilité de tirer parti de plusieurs partitions tout en conservant l’ordre des messages et la demande s’affichera plus tôt dans notre tableau de bord de journalisation. Dans certains scénarios, aucune réponse valide n’est jamais envoyée au hub d’événements, probablement en raison d’une erreur de demande irrécupérable dans le service Gestion des API, mais nous conservons un enregistrement de la demande.

La stratégie d’envoi du message de réponse HTTP étant très similaire à la demande, la configuration de la stratégie terminée ressemble à ce qui suit :

```xml
<policies>
  <inbound>
      <set-variable name="message-id" value="@(Guid.NewGuid())" />
      <log-to-eventhub logger-id="conferencelogger" partition-id="0">
      @{
          var requestLine = string.Format("{0} {1} HTTP/1.1\r\n",
                                                      context.Request.Method,
                                                      context.Request.Url.Path + context.Request.Url.QueryString);

          var body = context.Request.Body?.As<string>(true);
          if (body != null && body.Length > 1024)
          {
              body = body.Substring(0, 1024);
          }

          var headers = context.Request.Headers
                               .Where(h => h.Key != "Authorization" && h.Key != "Ocp-Apim-Subscription-Key")
                               .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                               .ToArray<string>();

          var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

          return "request:"   + context.Variables["message-id"] + "\n"
                              + requestLine + headerString + "\r\n" + body;
      }
  </log-to-eventhub>
  </inbound>
  <backend>
      <forward-request follow-redirects="true" />
  </backend>
  <outbound>
      <log-to-eventhub logger-id="conferencelogger" partition-id="1">
      @{
          var statusLine = string.Format("HTTP/1.1 {0} {1}\r\n",
                                              context.Response.StatusCode,
                                              context.Response.StatusReason);

          var body = context.Response.Body?.As<string>(true);
          if (body != null && body.Length > 1024)
          {
              body = body.Substring(0, 1024);
          }

          var headers = context.Response.Headers
                                          .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                                          .ToArray<string>();

          var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

          return "response:"  + context.Variables["message-id"] + "\n"
                              + statusLine + headerString + "\r\n" + body;
     }
  </log-to-eventhub>
  </outbound>
</policies>
```

La stratégie `set-variable` crée une valeur accessible à la fois par la stratégie `log-to-eventhub` de la section `<inbound>` et la section `<outbound>`.

## <a name="receiving-events-from-event-hubs"></a>Réception d’événements de hubs d’événements
Des événements sont reçus du hub d’événements Azure à l’aide du [protocole AMQP](https://www.amqp.org/). L’équipe de Microsoft Service Bus met à disposition les bibliothèques client pour faciliter l’utilisation des événements. Il existe deux approches différentes de prise en charge, l’une par un *consommateur Direct* et l’autre utilisant la classe `EventProcessorHost`. Vous trouverez des exemples de ces deux approches dans les [Guide de programmation des hubs d’événements](../event-hubs/event-hubs-programming-guide.md). La version courte des différences est : `Direct Consumer` vous donne un contrôle complet et le `EventProcessorHost` effectue une partie du travail pour vous, mais fait certaines hypothèses sur la façon dont vous traitez ces événements.

### <a name="eventprocessorhost"></a>EventProcessorHost
Dans cet exemple, nous utilisons `EventProcessorHost` par souci de simplicité ; cependant, cela n’est peut-être pas le meilleur choix dans ce scénario particulier. `EventProcessorHost` effectue le travail difficile qui consiste à s’assurer que n’avez pas à vous soucier des problèmes de threading dans une classe particulière de processeur d’événements. Cependant, dans notre scénario, nous nous contentons de convertir le message vers un autre format, et de le transférer vers un autre service à l’aide d’une méthode asynchrone. Il est inutile de mettre à jour l’état partagé et par conséquent, il n’y a aucun risque de problème lié aux threads. Pour la plupart des scénarios, `EventProcessorHost` est probablement le meilleur choix et c’est certainement l’option la plus facile.

### <a name="ieventprocessor"></a>IEventProcessor
Le concept central de l’utilisation de `EventProcessorHost` consiste à créer une implémentation de l’interface `IEventProcessor`, qui contient la méthode `ProcessEventAsync`. La fondation de cette méthode est indiquée ici :

```csharp
async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
{

    foreach (EventData eventData in messages)
    {
        _Logger.LogInfo(string.Format("Event received from partition: {0} - {1}", context.Lease.PartitionId,eventData.PartitionKey));

        try
        {
            var httpMessage = HttpMessage.Parse(eventData.GetBodyStream());
            await _MessageContentProcessor.ProcessHttpMessage(httpMessage);
        }
        catch (Exception ex)
        {
            _Logger.LogError(ex.Message);
        }
    }
    ... checkpointing code snipped ...
}
```

Une liste d’objets EventData est transmise à la méthode et nous exécutons une itération sur cette liste. Les octets de chaque méthode sont analysés dans un objet HttpMessage et cet objet est transmis à une instance de IHttpMessageProcessor.

### <a name="httpmessage"></a>HttpMessage
L’instance de `HttpMessage` contient trois éléments de données :

```csharp
public class HttpMessage
{
    public Guid MessageId { get; set; }
    public bool IsRequest { get; set; }
    public HttpRequestMessage HttpRequestMessage { get; set; }
    public HttpResponseMessage HttpResponseMessage { get; set; }

... parsing code snipped ...

}
```

L’instance `HttpMessage` contient une GUID `MessageId` qui vous permet de connecter la requête HTTP à la réponse HTTP correspondante et une valeur booléenne qui indique si l’objet contient une instance de HttpRequestMessage et HttpResponseMessage. En utilisant les classes intégrées HTTP de `System.Net.Http`, j’ai été capable de tirer parti du code d’analyse `application/http` inclus dans `System.Net.Http.Formatting`.  

### <a name="ihttpmessageprocessor"></a>IHttpMessageProcessor
L’instance `HttpMessage` est ensuite transmise à l’implémentation de `IHttpMessageProcessor`, qui est une interface que j’ai créée pour découpler la réception et l’interprétation de l’événement d’Azure Event Hubs et du traitement réel de celui-ci.

## <a name="forwarding-the-http-message"></a>Transfert du message HTTP
Pour cet exemple, j’ai décidé qu’il serait intéressant d’envoyer la requête HTTP via [Moesif API Analytics](https://www.moesif.com). Moesif est un service cloud spécialisé dans l’analyse et le débogage HTTP. Ils ont un niveau gratuit, de sorte qu’il est facile d’essayer et cela nous permet de voir les requêtes HTTP en temps réel circulant dans notre service de gestion des API.

L’implémentation `IHttpMessageProcessor` ressemble à ce qui suit,

```csharp
public class MoesifHttpMessageProcessor : IHttpMessageProcessor
{
    private readonly string RequestTimeName = "MoRequestTime";
    private MoesifApiClient _MoesifClient;
    private ILogger _Logger;
    private string _SessionTokenKey;
    private string _ApiVersion;
    public MoesifHttpMessageProcessor(ILogger logger)
    {
        var appId = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-APP-ID", EnvironmentVariableTarget.Process);
        _MoesifClient = new MoesifApiClient(appId);
        _SessionTokenKey = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-SESSION-TOKEN", EnvironmentVariableTarget.Process);
        _ApiVersion = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-API-VERSION", EnvironmentVariableTarget.Process);
        _Logger = logger;
    }

    public async Task ProcessHttpMessage(HttpMessage message)
    {
        if (message.IsRequest)
        {
            message.HttpRequestMessage.Properties.Add(RequestTimeName, DateTime.UtcNow);
            return;
        }

        EventRequestModel moesifRequest = new EventRequestModel()
        {
            Time = (DateTime) message.HttpRequestMessage.Properties[RequestTimeName],
            Uri = message.HttpRequestMessage.RequestUri.OriginalString,
            Verb = message.HttpRequestMessage.Method.ToString(),
            Headers = ToHeaders(message.HttpRequestMessage.Headers),
            ApiVersion = _ApiVersion,
            IpAddress = null,
            Body = message.HttpRequestMessage.Content != null ? System.Convert.ToBase64String(await message.HttpRequestMessage.Content.ReadAsByteArrayAsync()) : null,
            TransferEncoding = "base64"
        };

        EventResponseModel moesifResponse = new EventResponseModel()
        {
            Time = DateTime.UtcNow,
            Status = (int) message.HttpResponseMessage.StatusCode,
            IpAddress = Environment.MachineName,
            Headers = ToHeaders(message.HttpResponseMessage.Headers),
            Body = message.HttpResponseMessage.Content != null ? System.Convert.ToBase64String(await message.HttpResponseMessage.Content.ReadAsByteArrayAsync()) : null,
            TransferEncoding = "base64"
        };

        Dictionary<string, string> metadata = new Dictionary<string, string>();
        metadata.Add("ApimMessageId", message.MessageId.ToString());

        EventModel moesifEvent = new EventModel()
        {
            Request = moesifRequest,
            Response = moesifResponse,
            SessionToken = _SessionTokenKey != null ? message.HttpRequestMessage.Headers.GetValues(_SessionTokenKey).FirstOrDefault() : null,
            Tags = null,
            UserId = null,
            Metadata = metadata
        };

        Dictionary<string, string> response = await _MoesifClient.Api.CreateEventAsync(moesifEvent);

        _Logger.LogDebug("Message forwarded to Moesif");
    }

    private static Dictionary<string, string> ToHeaders(HttpHeaders headers)
    {
        IEnumerable<KeyValuePair<string, IEnumerable<string>>> enumerable = headers.GetEnumerator().ToEnumerable();
        return enumerable.ToDictionary(p => p.Key, p => p.Value.GetEnumerator()
                                                         .ToEnumerable()
                                                         .ToList()
                                                         .Aggregate((i, j) => i + ", " + j));
    }
}
```

Le `MoesifHttpMessageProcessor` tire parti d’une [bibliothèque d’API C# pour Moesif](https://www.moesif.com/docs/api?csharp#events) qui facilite l’envoi des données d’événement HTTP dans leur service. Pour envoyer des données HTTP à l’API Moesif Collector, vous devez avoir un compte et un ID d’application. Vous obtenez un ID d’application Moesif en créant un compte sur le [site Web de Moesif](https://www.moesif.com), puis en accédant au _menu en haut à droite_ -> _Programme d’installation de l’application_.

## <a name="complete-sample"></a>Exemple complet
Le [code source](https://github.com/dgilling/ApimEventProcessor) et les tests de l’exemple se trouvent sur GitHub. Vous avez besoin d’un [service de gestion des API](get-started-create-service-instance.md), [d’un Event Hub connecté](api-management-howto-log-event-hubs.md), et d’un [compte de stockage](../storage/common/storage-account-create.md) pour exécuter l’exemple vous-même.   

L’exemple est une simple application console qui écoute les événements provenant d’Event Hub, les convertit en objets Moesif `EventRequestModel` et `EventResponseModel`, puis les transmet à l’API Moesif Collector.

Dans l’image animée suivante, vous pouvez voir une demande adressée à une API dans le portail des développeurs, l’application console affichant le message reçu, traité et transmis, puis les demande et réponse affichées dans le flux d’événements.

![Démonstration de la requête transmise à Runscope](./media/api-management-log-to-eventhub-sample/apim-eventhub-runscope.gif)

## <a name="summary"></a>Résumé
Le service Gestion des API Azure fournit un emplacement idéal pour capturer le trafic HTTP qui circule vers et depuis vos API. Azure Event Hubs est une solution pouvant être mise à l’échelle et économique permettant de capturer le trafic et de l’intégrer à des systèmes de traitement secondaire pour la journalisation, la surveillance et d’autres analyses sophistiquées. La connexion à des systèmes de surveillance de trafic tiers tels que Moesif se résume à la rédaction de quelques dizaines de lignes de code.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur Azure Event Hubs
  * [Prise en main avec Azure Event Hubs](../event-hubs/event-hubs-c-getstarted-send.md)
  * [Réception de messages avec EventProcessorHost](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md)
  * [Guide de programmation Event Hubs](../event-hubs/event-hubs-programming-guide.md)
* En savoir plus sur l’intégration de Gestion des API et Event Hubs
  * [Comment enregistrer des événements sur Azure Event Hubs dans Gestion des API Azure](api-management-howto-log-event-hubs.md)
  * [Référence d’entité d’enregistreur](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-logger-entity)
  * [Référence de stratégie log-to-eventhub](./api-management-advanced-policies.md#log-to-eventhub)
