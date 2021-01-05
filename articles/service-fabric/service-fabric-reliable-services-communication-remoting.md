---
title: Communication à distance des services à l’aide de C# dans Service Fabric
description: La communication à distance dans Service Fabric permet aux clients et aux services de communiquer avec les services C# en utilisant un appel de procédure distante.
ms.topic: conceptual
ms.date: 09/20/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: c3659fea73abae3c9c5264f227b90d0af95a93e7
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576653"
---
# <a name="service-remoting-in-c-with-reliable-services"></a>Communication à distance des services en C# avec Reliable Services

> [!div class="op_single_selector"]
> * [C# sur Windows](service-fabric-reliable-services-communication-remoting.md)
> * [Java sur Linux](service-fabric-reliable-services-communication-remoting-java.md)
>
>

Pour les services qui ne sont pas liés à une pile ou à un protocole de communication particulier, comme API web, Windows Communication Foundation ou autres, l’infrastructure Reliable Services fournit un mécanisme de communication à distance pour configurer rapidement et facilement des appels de procédure distante pour les services. Cet article explique comment configurer les appels de procédure distante pour les services écrits en C#.

## <a name="set-up-remoting-on-a-service"></a>Configuration de la communication à distance sur un service

Vous pouvez configurer une communication à distance pour un service en deux étapes simples :

1. Créez une interface pour implémenter votre service. Cette interface définit les méthodes disponibles pour un appel de procédure distante sur votre service. Ces méthodes doivent être des méthodes asynchrones retournant des tâches. L'interface doit implémenter `Microsoft.ServiceFabric.Services.Remoting.IService` pour signaler que le service dispose d'une interface de communication à distance.
2. Utilisez un écouteur de communication à distance dans votre service. Un écouteur de communication à distance est une implémentation de `ICommunicationListener` qui fournit des fonctionnalités de communication à distance. L’espace de noms `Microsoft.ServiceFabric.Services.Remoting.Runtime` contient la méthode d’extension `CreateServiceRemotingInstanceListeners` pour les services avec et sans état qui peuvent être utilisés pour créer un écouteur de communication à distance à l’aide du protocole de transport de communication à distance par défaut.

>[!NOTE]
>L’espace de noms `Remoting` est disponible sous la forme d’un package NuGet distinct appelé `Microsoft.ServiceFabric.Services.Remoting`.

Par exemple, le service sans état suivant expose une méthode unique pour obtenir « Hello World » sur un appel de procédure distante :

```csharp
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Remoting;
using Microsoft.ServiceFabric.Services.Remoting.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

public interface IMyService : IService
{
    Task<string> HelloWorldAsync();
}

class MyService : StatelessService, IMyService
{
    public MyService(StatelessServiceContext context)
        : base (context)
    {
    }

    public Task<string> HelloWorldAsync()
    {
        return Task.FromResult("Hello!");
    }

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
     return this.CreateServiceRemotingInstanceListeners();
    }
}
```

> [!NOTE]
> Les arguments et les types de retour dans l’interface de service peuvent être de type simple, complexe ou personnalisé, mais ils doivent être sérialisables par l’élément .NET [DataContractSerializer](/dotnet/framework/wcf/feature-details/types-supported-by-the-data-contract-serializer).
>
>

## <a name="call-remote-service-methods"></a>Méthodes d’appel de service distant

L’appel de méthodes sur un service à l’aide de la pile de communication à distance est effectué avec un proxy local pour le service via la classe `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` . La méthode `ServiceProxy` crée un proxy local à l’aide de l’interface implémentée par le service. Avec ce proxy, vous pouvez appeler à distance des méthodes sur l’interface.

```csharp

IMyService helloWorldClient = ServiceProxy.Create<IMyService>(new Uri("fabric:/MyApplication/MyHelloWorldService"));

string message = await helloWorldClient.HelloWorldAsync();

```

L'infrastructure de communication à distance propage les exceptions levées par le service au client. Par conséquent, en cas d’utilisation de `ServiceProxy`, le client est responsable de la prise en charge des exceptions levées par le service.

## <a name="service-proxy-lifetime"></a>Durée de vie du proxy du service

La création de proxy de service étant une opération légère, vous pouvez en créer autant de fois que nécessaire. Les instances de proxy de service peuvent être réutilisées tant qu’elles sont nécessaires. Si un appel de procédure distante lève une exception, vous pouvez toujours réutiliser la même instance de proxy. Chaque proxy de service contient un client de communication qui permet d’envoyer des messages sur le réseau. Lors de l’invocation d’appels distants, des vérifications en interne sont effectuées pour déterminer si le client de communication est valide. Selon les résultats de ces vérifications, le client de communication est recréé si nécessaire. Par conséquent, si une exception se produit, il est inutile de recréer `ServiceProxy`.

### <a name="service-proxy-factory-lifetime"></a>Durée de vie de fabrique de proxy de service

[ServiceProxyFactory](/dotnet/api/microsoft.servicefabric.services.remoting.client.serviceproxyfactory) est une fabrique qui crée des instances de proxy pour différentes interfaces de communication à distance. Si vous utilisez l’API `ServiceProxyFactory.CreateServiceProxy` pour créer un proxy, l’infrastructure crée un proxy de service singleton.
Il est utile d’en créer un manuellement lorsque vous devez remplacer les propriétés [IServiceRemotingClientFactory](/dotnet/api/microsoft.servicefabric.services.remoting.v1.client.iserviceremotingclientfactory).
La création de fabrique est une opération coûteuse. Une fabrique de proxy de service conserve le cache interne du client de communication.
Il est recommandé de mettre en cache la fabrique de proxy de service aussi longtemps que possible.

## <a name="remoting-exception-handling"></a>Gestion des exceptions à distance

Toutes les exceptions à distance levées par l’API de service sont envoyées au client en tant que AggregateException. Les exceptions à distance doivent être sérialisables par DataContract. Si elles ne le sont pas, l’API de proxy lève une exception [ServiceException](/dotnet/api/microsoft.servicefabric.services.communication.serviceexception) contenant l’erreur de sérialisation.

Le proxy de service gère toutes les exceptions de basculement pour la partition pour laquelle il est créé. Il résout à nouveau les points de terminaison s’il existe des exceptions de basculement (exceptions non temporaires) et retente l’appel avec le point de terminaison correct. Le nombre de nouvelles tentatives pour des exceptions de basculement est indéfini.
En cas d’exceptions transitoires, le proxy retente l’appel.

Les paramètres de nouvelle tentative par défaut sont fournis par [OperationRetrySettings](/dotnet/api/microsoft.servicefabric.services.communication.client.operationretrysettings).

Un utilisateur peut configurer ces valeurs en passant l’objet OperationRetrySettings au constructeur de ServiceProxyFactory.

## <a name="use-the-remoting-v2-stack"></a>Utilisation de la pile V2 de communication à distance

Depuis la version 2.8 du package NuGet de communication à distance, vous avez la possibilité d’utiliser la pile V2 de communication à distance. La pile V2 de communication à distance est plus performante. Elle fournit également des fonctionnalités telles que la sérialisation personnalisée et des API plus enfichables.
Le code du modèle continue d’utiliser la pile V1 de communication à distance.
La pile V2 de communication à distance n’est pas compatible avec la pile V1 qui l’a précédée. Suivez les instructions de l’article [Mise à niveau de V1 à V2](#upgrade-from-remoting-v1-to-remoting-v2) afin d’éviter des effets sur la disponibilité du service.

Les approches suivantes sont disponibles pour activer la pile V2.

### <a name="use-an-assembly-attribute-to-use-the-v2-stack"></a>Utilisation d’un attribut d’assembly pour utiliser la pile V2

Ces étapes modifient le code du modèle pour utiliser la pile V2 à l’aide d’un attribut d’assembly.

1. Modifiez la ressource de point de terminaison de `"ServiceEndpoint"` à `"ServiceEndpointV2"` dans le manifeste de service.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2" />
    </Endpoints>
   </Resources>
   ```

2. Utilisez la méthode d'extension `Microsoft.ServiceFabric.Services.Remoting.Runtime.CreateServiceRemotingInstanceListeners` pour créer des écouteurs de la communication à distance (identiques pour V1 et V2).

   ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
   ```

3. Marquez l’assembly contenant les interfaces de communication à distance avec un attribut `FabricTransportServiceRemotingProvider`.

   ```csharp
   [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2, RemotingClientVersion = RemotingClientVersion.V2)]
   ```

Aucune modification du code n’est requise dans le projet client.
Générez l’assembly client avec l’assembly de l’interface, pour faire en sorte que l’attribut d’assembly illustré ci-dessus soit utilisé.

### <a name="use-explicit-v2-classes-to-use-the-v2-stack"></a>Utilisation de classes V2 explicites pour utiliser la pile V2

Au lieu d’utiliser un attribut d’assembly, la pile V2 peut également être activée à l’aide de classes V2 explicites.

Ces étapes modifient le code du modèle pour utiliser la pile V2 avec des classes V2 explicites.

1. Modifiez la ressource de point de terminaison de `"ServiceEndpoint"` à `"ServiceEndpointV2"` dans le manifeste de service.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2" />
    </Endpoints>
   </Resources>
   ```

2. Utilisez [FabricTransportServiceRemotingListener](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.runtime.fabrictransportserviceremotinglistener?view=azure-dotnet) à partir de l’espace de noms `Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Runtime`.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[]
        {
            new ServiceInstanceListener((c) =>
            {
                return new FabricTransportServiceRemotingListener(c, this);

            })
        };
    }
   ```

3. Utilisez [FabricTransportServiceRemotingClientFactory ](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.client.fabrictransportserviceremotingclientfactory?view=azure-dotnet) à partir de l’espace de noms `Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Client` pour créer des clients.

   ```csharp
   var proxyFactory = new ServiceProxyFactory((c) =>
          {
              return new FabricTransportServiceRemotingClientFactory();
          });
   ```

## <a name="upgrade-from-remoting-v1-to-remoting-v2"></a>Mise à niveau de la version V1 vers la version V2 de communication à distance

Pour procéder à la mise à niveau de la version V1 vers la version V2, des mises à niveau en 2 étapes sont requises. Suivez les étapes dans l’ordre indiqué.

1. Procédez à la mise à niveau du service V1 vers le service V2 à l’aide de l’attribut ci-après.
Cette modification permet de s’assurer que le service écoute sur les écouteurs V1 et V2.

    a. Ajoutez une ressource de point de terminaison nommée « ServiceEndpointV2 » au manifeste de service.
      ```xml
      <Resources>
        <Endpoints>
          <Endpoint Name="ServiceEndpointV2" />  
        </Endpoints>
      </Resources>
      ```

    b. Utilisez la méthode d’extension suivante pour créer un écouteur de communication à distance.

    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
    ```

    c. Ajoutez l’attribut d’assembly sur les interfaces de communication à distance pour utiliser les écouteurs V1 et V2, et le client V2.
    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2|RemotingListenerVersion.V1, RemotingClientVersion = RemotingClientVersion.V2)]

      ```
2. Mettez à niveau le client V1 vers un client V2 à l’aide de l’attribut de client V2.
Cette étape vérifie que le client utilise la pile V2.
Aucune modification du projet/service du client n’est requise. La création de projets de client avec l’assembly de l’interface mise à jour est suffisante.

3. Cette étape est facultative. Utilisez l’attribut d’écouteur V2, puis mettez à niveau le service V2.
Cette étape vérifie que le service écoute uniquement sur l’écouteur V2.

    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2, RemotingClientVersion = RemotingClientVersion.V2)]
    ```


## <a name="use-the-remoting-v2-interface-compatible-stack"></a>Utiliser la pile V2 de communication à distance (compatible avec l’interface)

 La pile de communication à distance V2 (compatible avec l’interface, appelée V2_1) a toutes les fonctionnalités de la pile de communication à distance V2. Sa pile d’interface est compatible avec la pile V1 de communication à distance, mais elle ne présente pas de compatibilité descendante avec les piles V1 et V2. Pour mettre à niveau de V1 à V2_1 sans affecter la disponibilité du service, suivez les étapes décrites dans l’article Mettre à niveau de la version V1 vers la version V2 (compatible avec l’interface).


### <a name="use-an-assembly-attribute-to-use-the-remoting-v2-interface-compatible-stack"></a>Utilisation d’un attribut d’assembly pour se servir de la pile V2 de communication à distance (compatible avec l’interface)

Pour passer à une pile V2_1, suivez ces étapes.

1. Ajoutez une ressource de point de terminaison nommée « ServiceEndpointV2_1 » au manifeste de service.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2_1" />  
    </Endpoints>
   </Resources>
   ```

2. Utilisez la méthode d’extension de communication à distance pour créer un écouteur de communication à distance.

   ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
   ```

3. Ajoutez un [attribut d’assembly](/dotnet/api/microsoft.servicefabric.services.remoting.fabrictransport.fabrictransportserviceremotingproviderattribute?view=azure-dotnet) sur les interfaces de communication à distance.

   ```csharp
    [assembly:  FabricTransportServiceRemotingProvider(RemotingListenerVersion=  RemotingListenerVersion.V2_1, RemotingClientVersion= RemotingClientVersion.V2_1)]

   ```

Aucune modification n’est requise dans le projet de client.
Générez l’assembly du client avec l’assembly de l’interface, pour être certain que l’attribut d’assembly précédent est utilisé.

### <a name="use-explicit-remoting-classes-to-create-a-listenerclient-factory-for-the-v2-interface-compatible-version"></a>Utilisation de classes de communication à distance explicites pour créer une fabrique d’écouteur/de client pour la version V2 (compatible avec l’interface)

Procédez comme suit :

1. Ajoutez une ressource de point de terminaison nommée « ServiceEndpointV2_1 » au manifeste de service.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2_1" />  
    </Endpoints>
   </Resources>
   ```

2. Utilisez l’[écouteur V2 de communication à distance](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.runtime.fabrictransportserviceremotinglistener?view=azure-dotnet). Le nom de la ressource de point de terminaison de service par défaut utilisé est « ServiceEndpointV2_1 ». Il doit être défini dans le manifeste de service.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[]
        {
            new ServiceInstanceListener((c) =>
            {
                var settings = new FabricTransportRemotingListenerSettings();
                settings.UseWrappedMessage = true;
                return new FabricTransportServiceRemotingListener(c, this,settings);

            })
        };
    }
   ```

3. Utilisez la [fabrique de client](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.client.fabrictransportserviceremotingclientfactory?view=azure-dotnet) V2.
   ```csharp
   var proxyFactory = new ServiceProxyFactory((c) =>
          {
            var settings = new FabricTransportRemotingSettings();
            settings.UseWrappedMessage = true;
            return new FabricTransportServiceRemotingClientFactory(settings);
          });
   ```

## <a name="upgrade-from-remoting-v1-to-remoting-v2-interface-compatible"></a>Mise à niveau de la version V1 vers la version V2 de communication à distance (compatible avec l’interface)

Pour mettre à niveau de la version V1 vers la version V2 (compatible avec l’interface, appelée V2_1), des mises à niveau en deux étapes sont nécessaires. Suivez les étapes dans l’ordre indiqué.

> [!NOTE]
> Lors de la mise à niveau de V1 vers V2, assurez-vous que l'espace de nom `Remoting` est mis à jour pour utiliser V2. Exemple : « Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Client »
>
>

1. Procédez à la mise à niveau du service V1 vers le service V2_1 à l’aide de l’attribut suivant.
Cette modification permet de s’assurer que le service écoute sur les écouteurs V1 et V2_1.

    a. Ajoutez une ressource de point de terminaison nommée « ServiceEndpointV2_1 » au manifeste de service.
      ```xml
      <Resources>
        <Endpoints>
          <Endpoint Name="ServiceEndpointV2_1" />  
        </Endpoints>
      </Resources>
      ```

    b. Utilisez la méthode d’extension suivante pour créer un écouteur de communication à distance.

    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
    ```

    c. Ajoutez l’attribut d’assembly sur les interfaces de communication à distance pour utiliser les écouteurs V1 et V2_1, et le client V2_1.
    ```csharp
   [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1 | RemotingListenerVersion.V1, RemotingClientVersion = RemotingClientVersion.V2_1)]

      ```
2. Procédez à la mise à niveau du client V1 vers le client V2_1 à l’aide de l’attribut de client V2_1.
Cette étape vérifie que le client utilise la pile V2_1.
Aucune modification du projet/service du client n’est requise. La création de projets de client avec l’assembly de l’interface mise à jour est suffisante.

3. Cette étape est facultative. Supprimez la version d’écouteur V1 de l’attribut, puis procédez à la mise à niveau vers le service V2.
Cette étape vérifie que le service écoute uniquement sur l’écouteur V2.

    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1, RemotingClientVersion = RemotingClientVersion.V2_1)]
    ```
  
### <a name="use-custom-serialization-with-a-remoting-wrapped-message"></a>Utilisation d’une sérialisation personnalisée avec un message encapsulé de communication à distance

Pour un message encapsulé de communication à distance, nous créons un objet encapsulé unique contenant tous les paramètres sous forme d’un champ.
Procédez comme suit :

1. Implémentez l’interface `IServiceRemotingMessageSerializationProvider` pour fournir une implémentation pour une sérialisation personnalisée.
    Cet extrait de code montre à quoi ressemble l’implémentation.

      ```csharp
      public class ServiceRemotingJsonSerializationProvider : IServiceRemotingMessageSerializationProvider
      {
        public IServiceRemotingMessageBodyFactory CreateMessageBodyFactory()
        {
          return new JsonMessageFactory();
        }

        public IServiceRemotingRequestMessageBodySerializer CreateRequestMessageSerializer(Type serviceInterfaceType, IEnumerable<Type> requestWrappedType, IEnumerable<Type> requestBodyTypes = null)
        {
          return new ServiceRemotingRequestJsonMessageBodySerializer();
        }

        public IServiceRemotingResponseMessageBodySerializer CreateResponseMessageSerializer(Type serviceInterfaceType, IEnumerable<Type> responseWrappedType, IEnumerable<Type> responseBodyTypes = null)
        {
          return new ServiceRemotingResponseJsonMessageBodySerializer();
        }
      }
      ```
      ```csharp
        class JsonMessageFactory : IServiceRemotingMessageBodyFactory
            {

              public IServiceRemotingRequestMessageBody CreateRequest(string interfaceName, string methodName, int numberOfParameters, object wrappedRequestObject)
              {
                return new JsonBody(wrappedRequestObject);
              }

              public IServiceRemotingResponseMessageBody CreateResponse(string interfaceName, string methodName, object wrappedRequestObject)
              {
                return new JsonBody(wrappedRequestObject);
              }
            }
      ```
      ```csharp
      class ServiceRemotingRequestJsonMessageBodySerializer : IServiceRemotingRequestMessageBodySerializer
        {
            private JsonSerializer serializer;

            public ServiceRemotingRequestJsonMessageBodySerializer()
            {
              serializer = JsonSerializer.Create(new JsonSerializerSettings()
              {
                TypeNameHandling = TypeNameHandling.All
                });
              }

              public IOutgoingMessageBody Serialize(IServiceRemotingRequestMessageBody serviceRemotingRequestMessageBody)
             {
               if (serviceRemotingRequestMessageBody == null)
               {
                 return null;
               }          
               using (var writeStream = new MemoryStream())
               {
                 using (var jsonWriter = new JsonTextWriter(new StreamWriter(writeStream)))
                 {
                   serializer.Serialize(jsonWriter, serviceRemotingRequestMessageBody);
                   jsonWriter.Flush();
                   var bytes = writeStream.ToArray();
                   var segment = new ArraySegment<byte>(bytes);
                   var segments = new List<ArraySegment<byte>> { segment };
                   return new OutgoingMessageBody(segments);
                 }
               }
              }

              public IServiceRemotingRequestMessageBody Deserialize(IIncomingMessageBody messageBody)
             {
               using (var sr = new StreamReader(messageBody.GetReceivedBuffer()))
               {
                 using (JsonReader reader = new JsonTextReader(sr))
                 {
                   var ob = serializer.Deserialize<JsonBody>(reader);
                   return ob;
                 }
               }
             }
            }
      ```
      ```csharp
      class ServiceRemotingResponseJsonMessageBodySerializer : IServiceRemotingResponseMessageBodySerializer
       {
         private JsonSerializer serializer;

        public ServiceRemotingResponseJsonMessageBodySerializer()
        {
          serializer = JsonSerializer.Create(new JsonSerializerSettings()
          {
              TypeNameHandling = TypeNameHandling.All
            });
          }

          public IOutgoingMessageBody Serialize(IServiceRemotingResponseMessageBody responseMessageBody)
          {
            if (responseMessageBody == null)
            {
              return null;
            }

            using (var writeStream = new MemoryStream())
            {
              using (var jsonWriter = new JsonTextWriter(new StreamWriter(writeStream)))
              {
                serializer.Serialize(jsonWriter, responseMessageBody);
                jsonWriter.Flush();
                var bytes = writeStream.ToArray();
                var segment = new ArraySegment<byte>(bytes);
                var segments = new List<ArraySegment<byte>> { segment };
                return new OutgoingMessageBody(segments);
              }
            }
          }

          public IServiceRemotingResponseMessageBody Deserialize(IIncomingMessageBody messageBody)
          {

             using (var sr = new StreamReader(messageBody.GetReceivedBuffer()))
             {
               using (var reader = new JsonTextReader(sr))
               {
                 var obj = serializer.Deserialize<JsonBody>(reader);
                 return obj;
               }
             }
           }
       }
    ```
    ```csharp
    class JsonBody : WrappedMessage, IServiceRemotingRequestMessageBody, IServiceRemotingResponseMessageBody
    {
          public JsonBody(object wrapped)
          {
            this.Value = wrapped;
          }

          public void SetParameter(int position, string parameName, object parameter)
          {  //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public object GetParameter(int position, string parameName, Type paramType)
          {
            //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public void Set(object response)
          { //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public object Get(Type paramType)
          {  //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }
    }
    ```

2. Remplacez le fournisseur de sérialisation par défaut par `JsonSerializationProvider` pour un écouteur de communication à distance.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
   {
       return new[]
       {
           new ServiceInstanceListener((c) =>
           {
               return new FabricTransportServiceRemotingListener(c, this,
                   new ServiceRemotingJsonSerializationProvider());
           })
       };
   }
   ```

3. Remplacez le fournisseur de sérialisation par défaut par `JsonSerializationProvider` pour une fabrique de clients de communication à distance.

    ```csharp
    var proxyFactory = new ServiceProxyFactory((c) =>
    {
        return new FabricTransportServiceRemotingClientFactory(
        serializationProvider: new ServiceRemotingJsonSerializationProvider());
      });
      ```

## <a name="next-steps"></a>Étapes suivantes

* [API Web avec OWIN dans Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md)
* [Communication de Windows Communication Foundation avec Reliable Services](service-fabric-reliable-services-communication-wcf.md)
* [Sécurisation des communications pour Reliable Services](service-fabric-reliable-services-secure-communication.md)
