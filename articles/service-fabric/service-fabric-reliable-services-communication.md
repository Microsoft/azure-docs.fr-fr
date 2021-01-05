---
title: Vue d’ensemble de la communication Reliable Services
description: Présentation du modèle de communication Reliable Services, notamment de l’ouverture d’écouteurs sur les services, de la résolution des points de terminaison et de la communication entre les services.
ms.topic: conceptual
ms.date: 11/01/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: e7dc10055633c8e6dd2c645f28b774d5d5f3ac3f
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574324"
---
# <a name="how-to-use-the-reliable-services-communication-apis"></a>Utilisation des API de communication de Reliable Services
Azure Service Fabric, en tant que plateforme, est totalement indépendant de la communication entre les services. Tous les protocoles et toutes les piles sont acceptables, de UDP à HTTP. C'est au développeur de services de choisir comment les services doivent communiquer. L’infrastructure d’application de Reliable Services fournit des piles de communication intégrées, ainsi que des API que vous pouvez utiliser pour générer vos composants de communication personnalisés.

## <a name="set-up-service-communication"></a>Configurer la communication de service
L'API Reliable Services utilise une interface simple pour la communication de service. Pour ouvrir un point de terminaison pour votre service, implémentez simplement cette interface :

```csharp

public interface ICommunicationListener
{
    Task<string> OpenAsync(CancellationToken cancellationToken);

    Task CloseAsync(CancellationToken cancellationToken);

    void Abort();
}

```

```java
public interface CommunicationListener {
    CompletableFuture<String> openAsync(CancellationToken cancellationToken);

    CompletableFuture<?> closeAsync(CancellationToken cancellationToken);

    void abort();
}
```

Vous pouvez ensuite ajouter votre implémentation d’écouteur de communication en la retournant dans une substitution de méthode de classe de base de services.

Pour des services sans état :

```csharp
public class MyStatelessService : StatelessService
{
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        ...
    }
    ...
}
```
```java
public class MyStatelessService extends StatelessService {

    @Override
    protected List<ServiceInstanceListener> createServiceInstanceListeners() {
        ...
    }
    ...
}
```

Pour des services avec état :

```java
    @Override
    protected List<ServiceReplicaListener> createServiceReplicaListeners() {
        ...
    }
    ...
```

```csharp
public class MyStatefulService : StatefulService
{
    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
        ...
    }
    ...
}
```

Dans les deux cas, vous retournez une collection d’écouteurs. Cela permet à votre service d’écouter sur plusieurs points de terminaison, utilisant éventuellement différents protocoles, à l’aide de plusieurs écouteurs. Par exemple, vous pouvez avoir un écouteur HTTP et un écouteur WebSocket distinct. Chaque écouteur obtient un nom et la collection qui résulte des paires *nom : adresse* est représentée en tant qu’objet JSON quand un client demande les adresses d’écoute pour une instance de service ou une partition.

Dans un service sans état, la substitution retourne une collection de ServiceInstanceListeners. Un `ServiceInstanceListener` contient une fonction permettant de créer un `ICommunicationListener(C#) / CommunicationListener(Java)` et de lui donner un nom. Pour les services avec état, la substitution retourne une collection de ServiceReplicaListener. Cela diffère légèrement de son homologue sans état, car un `ServiceReplicaListener` a une option pour ouvrir un `ICommunicationListener` sur des réplicas secondaires. Non seulement vous pouvez utiliser plusieurs écouteurs de communication dans un service, mais vous pouvez également spécifier ceux qui acceptent les demandes sur les réplicas secondaires, et ceux qui écoutent uniquement sur les réplicas principaux.

Par exemple, vous pouvez avoir un ServiceRemotingListener qui ne prend les appels RPC que sur les réplicas principaux, et un second écouteur personnalisé qui prend les demandes de lecture sur les réplicas secondaires sur HTTP :

```csharp
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new[]
    {
        new ServiceReplicaListener(context =>
            new MyCustomHttpListener(context),
            "HTTPReadonlyEndpoint",
            true),

        new ServiceReplicaListener(context =>
            this.CreateServiceRemotingListener(context),
            "rpcPrimaryEndpoint",
            false)
    };
}
```

> [!NOTE]
> Lors de la création de plusieurs écouteurs pour un service, vous **devez** affecter un nom unique à chaque écouteur.
>
>

Pour finir, décrivez les points de terminaison nécessaires pour le service dans le [manifeste de service](service-fabric-application-and-service-manifests.md) , dans la section consacrée aux points de terminaison.

```xml
<Resources>
    <Endpoints>
      <Endpoint Name="WebServiceEndpoint" Protocol="http" Port="80" />
      <Endpoint Name="OtherServiceEndpoint" Protocol="tcp" Port="8505" />
    <Endpoints>
</Resources>

```

L’écouteur de communication peut accéder aux ressources de point de terminaison qui lui sont allouées à partir du `CodePackageActivationContext` dans `ServiceContext`. L’écouteur peut alors commencer à écouter les demandes quand il est ouvert.

```csharp
var codePackageActivationContext = serviceContext.CodePackageActivationContext;
var port = codePackageActivationContext.GetEndpoint("ServiceEndpoint").Port;

```
```java
CodePackageActivationContext codePackageActivationContext = serviceContext.getCodePackageActivationContext();
int port = codePackageActivationContext.getEndpoint("ServiceEndpoint").getPort();

```

> [!NOTE]
> Les ressources de point de terminaison sont communes au package de service entier et elles sont allouées par Service Fabric à l’activation du package de service. Plusieurs réplicas de service hébergés dans le même ServiceHost peuvent partager le même port. Cela signifie que l'écouteur de communication doit prendre en charge le partage de port. La méthode recommandée consiste à configurer l’écouteur de communication afin qu’il utiliser l’ID de la partition et l’ID du réplica/de l’instance lors de la génération de l’adresse d’écoute.
>
>

### <a name="service-address-registration"></a>Enregistrement d’adresse de service
Un service système appelé *service d’attribution de noms* s’exécute sur les clusters Service Fabric. Le service d’attribution de noms est un bureau d’enregistrement pour les services et leurs adresses sur lesquelles chaque instance ou réplica du service écoute. Une fois l’exécution de la méthode `OpenAsync(C#) / openAsync(Java)` d’un `ICommunicationListener(C#) / CommunicationListener(Java)` terminée, la valeur retournée est enregistrée dans le service d’attribution de noms. Cette valeur publiée dans le service d’attribution de noms est une chaîne dont le contenu peut être absolument quelconque. Cette valeur de chaîne est ce que les clients voient quand ils demandent au service d’attribution de noms une adresse pour le service.

```csharp
public Task<string> OpenAsync(CancellationToken cancellationToken)
{
    EndpointResourceDescription serviceEndpoint = serviceContext.CodePackageActivationContext.GetEndpoint("ServiceEndpoint");
    int port = serviceEndpoint.Port;

    this.listeningAddress = string.Format(
                CultureInfo.InvariantCulture,
                "http://+:{0}/",
                port);

    this.publishAddress = this.listeningAddress.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);

    this.webApp = WebApp.Start(this.listeningAddress, appBuilder => this.startup.Invoke(appBuilder));

    // the string returned here will be published in the Naming Service.
    return Task.FromResult(this.publishAddress);
}
```
```java
public CompletableFuture<String> openAsync(CancellationToken cancellationToken)
{
    EndpointResourceDescription serviceEndpoint = serviceContext.getCodePackageActivationContext.getEndpoint("ServiceEndpoint");
    int port = serviceEndpoint.getPort();

    this.publishAddress = String.format("http://%s:%d/", FabricRuntime.getNodeContext().getIpAddressOrFQDN(), port);

    this.webApp = new WebApp(port);
    this.webApp.start();

    /* the string returned here will be published in the Naming Service.
     */
    return CompletableFuture.completedFuture(this.publishAddress);
}
```

Service Fabric fournit des API qui permettent aux clients et à d’autres services de demander cette adresse par nom de service. Ceci est important, car l’adresse du service n’est pas statique. Les services se déplacent dans le cluster à des fins d’équilibrage des ressources et de disponibilité. Ce mécanisme permet aux clients de résoudre l'adresse d'écoute pour un service.

> [!NOTE]
> Pour suivre une procédure pas à pas complète sur l’écriture d’un écouteur de communication consultez [Services Service Fabric Web API avec l’auto-hébergement](./service-fabric-reliable-services-communication-aspnetcore.md) pour C#, alors que pour Java vous pouvez écrire votre propre implémentation de serveur HTTP, consultez l’exemple d’application EchoServer sur https://github.com/Azure-Samples/service-fabric-java-getting-started.
>
>

## <a name="communicating-with-a-service"></a>Communication avec un service
L’API Reliable Services fournit les bibliothèques suivantes pour l’écriture de clients qui communiquent avec des services.

### <a name="service-endpoint-resolution"></a>Résolution de point de terminaison de service
La première étape de la communication avec un service consiste à résoudre une adresse de point de terminaison de la partition ou une instance du service avec lequel vous souhaitez communiquer. La classe utilitaire `ServicePartitionResolver(C#) / FabricServicePartitionResolver(Java)` est une primitive de base qui aide les clients à déterminer le point de terminaison d’un service au moment de l’exécution. Dans la terminologie Service Fabric, le processus de détermination du point de terminaison d’un service est appelé *résolution de point de terminaison de service*.

Pour vous connecter à des services dans un cluster, vous pouvez créer un ServicePartitionResolver à l’aide de paramètres par défaut. Il s’agit de l’utilisation recommandée pour la plupart des situations :

```csharp
ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();
```
```java
FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();
```

Pour vous connecter à des services figurant dans un autre cluster, vous pouvez créer un ServicePartitionResolver avec un ensemble de points de terminaison de passerelle de cluster. Notez que les points de terminaison de passerelle sont simplement des points de terminaison différents pour la connexion au même cluster. Par exemple :

```csharp
ServicePartitionResolver resolver = new  ServicePartitionResolver("mycluster.cloudapp.azure.com:19000", "mycluster.cloudapp.azure.com:19001");
```
```java
FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver("mycluster.cloudapp.azure.com:19000", "mycluster.cloudapp.azure.com:19001");
```

En guise d’alternative, un `ServicePartitionResolver` peut recevoir une fonction de création d’un `FabricClient` à utiliser en interne :

```csharp
public delegate FabricClient CreateFabricClientDelegate();
```
```java
public FabricServicePartitionResolver(CreateFabricClient createFabricClient) {
...
}

public interface CreateFabricClient {
    public FabricClient getFabricClient();
}
```

`FabricClient` est l’objet utilisé pour communiquer avec le cluster Service Fabric pour diverses opérations de gestion sur le cluster. Cela est utile quand vous souhaitez contrôler davantage la façon dont un programme de résolution de partition de service interagit avec votre cluster. `FabricClient` effectue la mise en cache en interne et sa création est généralement coûteuse. Il est donc important de réutiliser les instances `FabricClient` autant que possible.

```csharp
ServicePartitionResolver resolver = new  ServicePartitionResolver(() => CreateMyFabricClient());
```
```java
FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver(() -> new CreateFabricClientImpl());
```

Une méthode de résolution est ensuite utilisée pour récupérer l’adresse d’un service ou d’une partition de service pour les services partitionnés.

```csharp
ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();

ResolvedServicePartition partition =
    await resolver.ResolveAsync(new Uri("fabric:/MyApp/MyService"), new ServicePartitionKey(), cancellationToken);
```
```java
FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();

CompletableFuture<ResolvedServicePartition> partition =
    resolver.resolveAsync(new URI("fabric:/MyApp/MyService"), new ServicePartitionKey());
```

Un ServicePartitionResolver peut facilement résoudre une adresse de service, mais il faut davantage de travail pour s’assurer que l’adresse résolue est correctement utilisable. Votre client doit détecter si la tentative de connexion a échoué en raison d’une erreur temporaire et peut être retentée (par exemple, le service a changé d’emplacement ou est temporairement indisponible), ou d’une erreur permanente (par exemple, le service a été supprimé ou la ressource demandée n’existe plus). Des instances de service ou réplicas peuvent passer d’un nœud à l’autre à tout moment pour plusieurs raisons. L’adresse de service résolue par le biais de ServicePartitionResolver peut être obsolète au moment où votre code client tente de se connecter. Dans ce cas encore, le client doit à nouveau résoudre l’adresse. Le code `ResolvedServicePartition` précédent indique que le programme de résolution doit effectuer une nouvelle tentative au lieu de simplement récupérer une adresse en mémoire cache.

Généralement, le code client ne doit pas fonctionner directement avec ServicePartitionResolver. Il est créé et transmis à des fabriques de clients de communication dans l’API Reliable Services. Les fabriques utilisent le programme de résolution en interne pour générer un objet client utilisable pour communiquer avec les services.

### <a name="communication-clients-and-factories"></a>Fabriques et clients de communication
La bibliothèque de fabrique de communication implémente un modèle standard de nouvelle tentative de gestion des erreurs, qui facilite les nouvelles tentatives de connexion aux points de terminaison de service résolus. La bibliothèque de fabrique fournit le mécanisme de nouvelle tentative, tandis que vous fournissez les gestionnaires d’erreurs.

`ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` définit l’interface de base implémentée par une fabrique de clients de communication qui produit des clients capables de communiquer avec un service Service Fabric. L’implémentation de CommunicationClientFactory dépend de la pile de communication utilisée par le service Service Fabric avec lequel le client veut communiquer. L’API Reliable Services fournit un `CommunicationClientFactoryBase<TCommunicationClient>`. Celui-ci assure une implémentation de base de l’interface CommunicationClientFactory et effectue des tâches communes à toutes les piles de communication. (ces tâches incluent l’utilisation d’un ServicePartitionResolver pour déterminer le point de terminaison de service). Les clients implémentent généralement la classe abstraite CommunicationClientFactoryBase pour gérer la logique spécifique de la pile de communication.

Le client de communication reçoit simplement une adresse qu’il utilise pour se connecter à un service. Le client peut utiliser n’importe quel protocole.

```csharp
public class MyCommunicationClient : ICommunicationClient
{
    public ResolvedServiceEndpoint Endpoint { get; set; }

    public string ListenerName { get; set; }

    public ResolvedServicePartition ResolvedServicePartition { get; set; }
}
```
```java
public class MyCommunicationClient implements CommunicationClient {

    private ResolvedServicePartition resolvedServicePartition;
    private String listenerName;
    private ResolvedServiceEndpoint endPoint;

    /*
     * Getters and Setters
     */
}
```

La fabrique de clients est principalement responsable de la création de clients de communication. Pour les clients qui ne maintiennent de connexion permanente, tel un client HTTP, la fabrique doit uniquement créer et retourner le client. D’autres protocoles qui maintiennent une connexion permanente, tels certains protocoles binaires, doivent également être validés par la fabrique pour déterminer si la connexion doit être recréée.  

```csharp
public class MyCommunicationClientFactory : CommunicationClientFactoryBase<MyCommunicationClient>
{
    protected override void AbortClient(MyCommunicationClient client)
    {
    }

    protected override Task<MyCommunicationClient> CreateClientAsync(string endpoint, CancellationToken cancellationToken)
    {
    }

    protected override bool ValidateClient(MyCommunicationClient clientChannel)
    {
    }

    protected override bool ValidateClient(string endpoint, MyCommunicationClient client)
    {
    }
}
```
```java
public class MyCommunicationClientFactory extends CommunicationClientFactoryBase<MyCommunicationClient> {

    @Override
    protected boolean validateClient(MyCommunicationClient clientChannel) {
    }

    @Override
    protected boolean validateClient(String endpoint, MyCommunicationClient client) {
    }

    @Override
    protected CompletableFuture<MyCommunicationClient> createClientAsync(String endpoint) {
    }

    @Override
    protected void abortClient(MyCommunicationClient client) {
    }
}
```

Enfin, un gestionnaire d’exceptions doit déterminer l’action à exécuter si une exception se produit. Les exceptions sont classées en **reproductibles** et **non reproductibles**.

* Les exceptions **Non reproductibles** sont simplement renvoyées à l’appelant.
* Les exceptions **reproductibles** sont encore classées en **temporaires** et **non temporaires**.
  * **temporaires** sont celles qui peuvent simplement faire l’objet d’une nouvelle tentative, sans nouvelle résolution de l’adresse du point de terminaison de service. Elles incluent notamment des problèmes réseau temporaires ou des réponses d’erreur de service autres que celles indiquant que l’adresse du point de terminaison de service est inexistante.
  * **Non-transient** sont celles qui nécessitent une nouvelle résolution de l’adresse du point de terminaison de service. Elles incluent notamment les exceptions indiquant que le point de terminaison de service n’a pas pu être atteint, ce qui signifie que le service s’est déplacé vers un autre nœud.

La fonction `TryHandleException` prend une décision concernant une exception donnée. Si elle **ne sait pas** quelle décision prendre à propos d’une exception, elle doit retourner la valeur **false**. Si elle **sait** quelle décision prendre, elle doit définir le résultat en conséquence et retourner la valeur **true**.

```csharp
class MyExceptionHandler : IExceptionHandler
{
    public bool TryHandleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings, out ExceptionHandlingResult result)
    {
        // if exceptionInformation.Exception is known and is transient (can be retried without re-resolving)
        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, true, retrySettings, retrySettings.DefaultMaxRetryCount);
        return true;


        // if exceptionInformation.Exception is known and is not transient (indicates a new service endpoint address must be resolved)
        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, false, retrySettings, retrySettings.DefaultMaxRetryCount);
        return true;

        // if exceptionInformation.Exception is unknown (let the next IExceptionHandler attempt to handle it)
        result = null;
        return false;
    }
}
```
```java
public class MyExceptionHandler implements ExceptionHandler {

    @Override
    public ExceptionHandlingResult handleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings) {

        /* if exceptionInformation.getException() is known and is transient (can be retried without re-resolving)
         */
        result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), true, retrySettings, retrySettings.getDefaultMaxRetryCount());
        return true;


        /* if exceptionInformation.getException() is known and is not transient (indicates a new service endpoint address must be resolved)
         */
        result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), false, retrySettings, retrySettings.getDefaultMaxRetryCount());
        return true;

        /* if exceptionInformation.getException() is unknown (let the next ExceptionHandler attempt to handle it)
         */
        result = null;
        return false;

    }
}
```
### <a name="putting-it-all-together"></a>Exemple complet
Avec des composants `ICommunicationClient(C#) / CommunicationClient(Java)`, `ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` et `IExceptionHandler(C#) / ExceptionHandler(Java)` construits autour d’un protocole de communication, un `ServicePartitionClient(C#) / FabricServicePartitionClient(Java)` encapsule le tout et fournit la boucle de gestion des erreurs et de résolution d’adresse de partition de service autour de ces composants.

```csharp
private MyCommunicationClientFactory myCommunicationClientFactory;
private Uri myServiceUri;

var myServicePartitionClient = new ServicePartitionClient<MyCommunicationClient>(
    this.myCommunicationClientFactory,
    this.myServiceUri,
    myPartitionKey);

var result = await myServicePartitionClient.InvokeWithRetryAsync(async (client) =>
   {
      // Communicate with the service using the client.
   },
   CancellationToken.None);

```
```java
private MyCommunicationClientFactory myCommunicationClientFactory;
private URI myServiceUri;

FabricServicePartitionClient myServicePartitionClient = new FabricServicePartitionClient<MyCommunicationClient>(
    this.myCommunicationClientFactory,
    this.myServiceUri,
    myPartitionKey);

CompletableFuture<?> result = myServicePartitionClient.invokeWithRetryAsync(client -> {
      /* Communicate with the service using the client.
       */
   });

```

## <a name="next-steps"></a>Étapes suivantes
* [ASP.NET Core avec Reliable Services](service-fabric-reliable-services-communication-aspnetcore.md)
* [Appels de procédure distante avec Reliable Services à distance](service-fabric-reliable-services-communication-remoting.md)
* [Communication WCF à l’aide de Reliable Services](service-fabric-reliable-services-communication-wcf.md)
