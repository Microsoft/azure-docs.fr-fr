---
title: Communication de service avec ASP.NET Core
description: Découvrez comment utiliser ASP.NET Core dans des applications Azure Service Fabric Reliable Services avec état et sans état.
ms.topic: conceptual
ms.date: 10/12/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: ba5626d477bbd6aa07d89703cc37b157f4cfd4d5
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576789"
---
# <a name="aspnet-core-in-azure-service-fabric-reliable-services"></a>ASP.NET Core dans le modèle Azure Reliable Services de Service Fabric

ASP.NET Core est une infrastructure open source et multiplateforme. Cette infrastructure est conçue pour générer des applications sur le cloud, connectées à internet, telles que les applications web, les applications IoT, et les backends mobiles.

Cet article constitue un guide détaillé pour héberger des services ASP.NET Core dans le modèle Reliable Services de Service Fabric à l’aide de l’ensemble de packages NuGet **Microsoft.ServiceFabric.AspNetCore.** .

Pour obtenir un didacticiel de présentation d’ASP.NET Core dans Service Fabric et des instructions de configuration de votre environnement de développement, consultez [Didacticiel : Créer et déployer une application avec un service frontal API Web ASP.NET Core et un service principal avec état](service-fabric-tutorial-create-dotnet-app.md).

Pour lire la suite de cet article, vous devez déjà maîtriser ASP.NET Core. Si ce n’est pas le cas, veuillez lire les [notions de base d’ASP.NET Core](/aspnet/core/fundamentals/index).

## <a name="aspnet-core-in-the-service-fabric-environment"></a>ASP.NET Core dans l’environnement Service Fabric

Les applications ASP.NET Core et Service Fabric peuvent s’exécuter sur .NET Core ou sur le .NET Framework complet. Vous pouvez utiliser l’infrastructure ASP.NET Core de deux manières dans Service Fabric :
 - **Hébergée en tant qu’exécutable invité**. Cette méthode est principalement utilisée pour exécuter des applications ASP.NET Core existantes sur Service Fabric sans modification de code.
 - **Exécutée dans un service Reliable Service**. Cette méthode permet des services ASP.NET Core avec état ainsi qu’une meilleure intégration au runtime Service Fabric.

La suite de cet article explique comment utiliser ASP.NET Core dans un service Reliable Service à l’aide des composants d’intégration ASP.NET Core fournis avec le Kit de développement logiciel (SDK) Service Fabric.

## <a name="service-fabric-service-hosting"></a>Hébergement du service Service Fabric

Dans Service Fabric, une ou plusieurs instances et/ou un ou plusieurs réplicas de votre service s’exécutent dans un *processus hôte de service*, un fichier exécutable qui exécute le code de votre service. En tant que créateur du service, vous possédez le processus hôte de service qui est activé et surveillé par Service Fabric à votre place.

Traditionnellement, ASP.NET (jusqu’à MVC 5) est étroitement lié à IIS via System.Web.dll. ASP.NET Core fournit une séparation entre le serveur web et votre application web. Cette séparation permet aux applications web d’être portables entre différents serveurs web. Elle permet également aux serveurs web d’être *auto-hébergés*. Cela signifie que vous pouvez démarrer un serveur web dans votre propre processus, par opposition à un processus appartenant à un logiciel de serveur web dédié, par exemple IIS.

Pour combiner un service Service Fabric et ASP.NET en tant qu’exécutable invité ou dans un service Reliable Service, vous devez être en mesure de démarrer ASP.NET dans votre processus hôte de service. L’auto-hébergement d’ASP.NET Core vous le permet.

## <a name="hosting-aspnet-core-in-a-reliable-service"></a>Hébergement d’ASP.NET Core dans un service Reliable Service
En règle générale, les applications ASP.NET Core auto-hébergées créent une méthode WebHost dans le point d’entrée d’une application, par exemple la méthode `static void Main()` dans `Program.cs`. Dans ce cas, le cycle de vie de la méthode WebHost est lié au cycle de vie du processus.

![Hébergement d’ASP.NET Core dans un processus][0]

Mais le point d’entrée de l’application n’est pas au bon endroit pour créer une méthode WebHost dans un service fiable. C’est parce que le point d’entrée de l’application est uniquement utilisé pour inscrire un type de service avec le runtime Service Fabric, afin qu’il puisse créer des instances de ce type de service. La méthode WebHost doit être créée dans un service Reliable Service. Dans le processus hôte de service, les instances de service et/ou les réplicas peuvent connaître plusieurs cycles de vie. 

Une instance du service Reliable Service est représentée par votre classe de service dérivant de `StatelessService` ou `StatefulService`. La pile de communication d’un service est contenue dans une implémentation `ICommunicationListener` dans votre classe de service. Les packages NuGet `Microsoft.ServiceFabric.AspNetCore.*` contiennent les implémentations de `ICommunicationListener`, qui démarrent et gèrent le WebHost ASP.NET Core pour Kestrel ou HTTP.sys dans un service Reliable Services.

![Diagramme pour l’hébergement d’ASP.NET Core dans un service Reliable Service][1]

## <a name="aspnet-core-icommunicationlisteners"></a>Écouteurs ICommunicationListeners d’ASP.NET Core
Les implémentations `ICommunicationListener` pour Kestrel et HTTP.sys dans les packages NuGet `Microsoft.ServiceFabric.AspNetCore.*` ont des modèles d’utilisation similaires. Mais elles effectuent des actions légèrement différentes propres à chaque serveur web. 

Les deux écouteurs de communications fournissent un constructeur qui accepte les arguments suivants :
 - **`ServiceContext serviceContext`**  : il s’agit de l’objet `ServiceContext` qui contient des informations sur le service en cours d’exécution.
 - **`string endpointName`**  : il s’agit du nom d’une configuration `Endpoint` dans le fichier ServiceManifest.xml. C’est là que se situe la principale différence entre les deux écouteurs de communication. HTTP.sys *nécessite* une configuration `Endpoint`, contrairement à Kestrel.
 - **`Func<string, AspNetCoreCommunicationListener, IWebHost> build`**  : il s’agit d’une expression lambda que vous implémentez dans laquelle vous créez et retournez un `IWebHost`. Vous pouvez ainsi configurer `IWebHost` comme vous le feriez normalement dans une application ASP.NET Core. L’expression lambda fournit une URL, qui est générée pour vous en fonction des options d’intégration de Service Fabric que vous utilisez et de la configuration `Endpoint` que vous fournissez. Vous pouvez ensuite modifier ou utiliser cette URL pour démarrer le serveur web.

## <a name="service-fabric-integration-middleware"></a>Intergiciel (middleware) d’intégration à Service Fabric
Le paquet NuGet `Microsoft.ServiceFabric.AspNetCore` inclut la méthode d’extension `UseServiceFabricIntegration` sur `IWebHostBuilder` qui ajoute l’intergiciel (middleware) prenant en charge Service Fabric. Cet intergiciel (middleware) configure l’élément Kestrel ou HTTP.sys `ICommunicationListener` pour inscrire une URL de service unique auprès du service d’affectation de noms de Service Fabric. Ensuite, il valide les demandes des clients pour garantir le fait que les clients se connectent au service approprié. 

Cette étape est nécessaire pour empêcher que les clients se connectent par erreur à un service incorrect. C’est parce que, dans un environnement hôte partagé comme Service Fabric, plusieurs applications web peuvent s’exécuter sur la même machine physique ou virtuelle, mais n’utilisent pas des noms d’hôte unique. Ce scénario est décrit plus en détail dans la section suivante.

### <a name="a-case-of-mistaken-identity"></a>Cas d’erreur d’identité
Quel que soit le protocole utilisé, les réplicas de service écoutent sur une combinaison IP:port unique. Lorsqu’un réplica de service a commencé à écouter sur un point de terminaison IP:port, il indique l’adresse de ce point de terminaison à Service Fabric Naming Service. Là, Il peut être détecté par des clients ou d’autres services. Si des services utilisent des ports d’application affectés de manière dynamique, un réplica de service peut utiliser par hasard le même point de terminaison IP:port d’un autre service qui existait précédemment sur la même machine physique ou virtuelle. Cela peut provoquer la connexion d’un client au mauvais service. Ce scénario peut se produire si la séquence d’événements suivante se produit :

 1. Le service A écoute sur 10.0.0.1:30000 via HTTP. 
 2. Le client résout le service A et obtient l’adresse 10.0.0.1:30000.
 3. Le service A passe à un autre nœud.
 4. Le service B est placé sur 10.0.0.1 et utilise par hasard le même port 30000.
 5. Le client essaie de se connecter au service A avec l’adresse mise en cache 10.0.0.1:30000.
 6. Le client est maintenant connecté au service B, sans détecter qu’il est connecté à un service incorrect.

Cela peut provoquer des bogues de façon aléatoire qui peuvent être difficiles à diagnostiquer.

### <a name="using-unique-service-urls"></a>Utilisation d’URL de service uniques
Pour éviter ces bogues, les services peuvent publier un point de terminaison dans Naming Service avec un identificateur unique, puis valider cet identificateur lors des demandes des clients. Il s’agit d’une action de coopération entre les services dans un environnement approuvé de client non hostile. Cette action ne permet pas d’authentification de service sécurisée dans un environnement de client hostile.

Dans un environnement approuvé, l’intergiciel (middleware) qui est ajouté par la méthode `UseServiceFabricIntegration` ajoute automatiquement un identificateur unique à l’adresse qui est publiée dans Naming Service. Il valide cet identificateur à chaque demande. Si l’identificateur ne correspond pas, l’intergiciel (middleware) retourne immédiatement une réponse HTTP 410 Supprimé.

Les services qui utilisent un port attribué dynamiquement doivent avoir recours à cet intergiciel (middleware).

Les services qui utilisent un port unique fixe ne connaissent pas ce problème dans un environnement coopératif. Un port de ce type est généralement utilisé pour les services externes qui nécessitent un port connu pour que les applications clientes s’y connectent. Par exemple, la plupart des applications web accessibles sur Internet utilisent les ports 80 ou 443 pour les connexions de navigateur web. Dans ce cas, l’identificateur unique ne doit pas être activé.

Le diagramme suivant illustre le flux de demande avec l’intergiciel (middleware) activé :

![Intégration d’ASP.NET Core à Service Fabric][2]

Les implémentations `ICommunicationListener` de Kestrel et HTTP.sys utilisent ce mécanisme de la même façon. Même si HTTP.sys peut différencier en interne les requêtes d’après leur chemin d’URL unique à l’aide de la fonctionnalité de partage de port **HTTP.sys** sous-jacente, cette fonctionnalité *n’est pas* utilisée par l’implémentation `ICommunicationListener` de HTTP.sys. Cela survient car cela entraîne des erreurs HTTP 503 et HTTP 404 dans le scénario décrit précédemment. Il est alors difficile pour les clients de déterminer l’objet de l’erreur, car les codes HTTP 503 et HTTP 404 sont couramment utilisés pour indiquer d’autres erreurs. 

Par conséquent, les implémentations `ICommunicationListener` Kestrel et HTTP.sys standardisent sur l’intergiciel (middleware) fourni par la méthode d’extension `UseServiceFabricIntegration`. Par conséquent, les clients n’ont qu’à effectuer une action de nouvelle résolution de point de terminaison de service sur les réponses HTTP 410.

## <a name="httpsys-in-reliable-services"></a>HTTP.sys dans Reliable Services
Vous pouvez utiliser HTTP.sys dans un service Reliable Services en important le package NuGet **Microsoft.ServiceFabric.AspNetCore.HttpSys**. Ce package contient `HttpSysCommunicationListener`, une implémentation de `ICommunicationListener`. `HttpSysCommunicationListener` vous permet de créer un WebHost ASP.NET Core dans un service Reliable Services utilisant HTTP.sys comme serveur web.

HTTP.sys est basé sur [l’API du serveur HTTP Windows](/windows/win32/http/http-api-start-page). Cet API utilise le pilote du noyau **HTTP.sys** pour traiter les requêtes HTTP et les acheminer vers les processus exécutant les applications web. De cette façon, plusieurs processus d’une même machine physique ou virtuelle peuvent héberger des applications web sur un même port, dont l’ambiguïté est levée par un chemin d’accès d’URL ou un nom d’hôte unique. Ces fonctionnalités sont utiles dans Service Fabric pour héberger plusieurs sites web dans un même cluster.

>[!NOTE]
>L’implémentation de HTTP.sys fonctionne uniquement sur la plateforme Windows.

Le diagramme suivant illustre l’utilisation du pilote de noyau **HTTP.sys** par HTTP.sys sur Windows pour le partage de port :

![Diagramme de HTTP.sys][3]

### <a name="httpsys-in-a-stateless-service"></a>HTTP.sys dans un service sans état
Pour utiliser `HttpSys` dans un service sans état, remplacez la méthode `CreateServiceInstanceListeners` et retournez une instance `HttpSysCommunicationListener` :

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext =>
            new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                new WebHostBuilder()
                    .UseHttpSys()
                    .ConfigureServices(
                        services => services
                            .AddSingleton<StatelessServiceContext>(serviceContext))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build()))
    };
}
```

### <a name="httpsys-in-a-stateful-service"></a>HTTP.sys dans un service avec état

Pour le moment, `HttpSysCommunicationListener` n’est pas conçu pour être utilisé dans des services avec état en raison de complications liées à la fonctionnalité de partage de port **HTTP.sys** sous-jacente. Pour plus d’informations, consultez la section suivante sur l’allocation de ports dynamiques avec HTTP.sys. Pour les services avec état, il est suggéré d’utiliser Kestrel comme serveur web.

### <a name="endpoint-configuration"></a>Configuration du point de terminaison

Une configuration `Endpoint` est nécessaire pour les serveurs web qui utilisent l’API du serveur HTTP Windows, y compris HTTP.sys. Les serveurs web qui utilisent cette API doivent d’abord réserver leur URL avec HTTP.sys (cela s’effectue normalement avec l’outil [netsh](/windows/win32/http/netsh-commands-for-http)). 

Cette action requiert des privilèges élevés dont vos services sont dépourvus par défaut. Les options « http » ou « https » de la propriété `Protocol` de la configuration `Endpoint` dans le fichier ServiceManifest.xml sont utilisées plus particulièrement pour indiquer au runtime Service Fabric d’inscrire une URL avec HTTP.sys pour votre compte. Cela est fait en utilisant le préfixe d’URL à [*caractère générique fort*](/windows/win32/http/urlprefix-strings).

Par exemple, pour réserver `http://+:80` pour un service, utilisez la configuration suivante dans le fichier ServiceManifest.xml :

```xml
<ServiceManifest ... >
    ...
    <Resources>
        <Endpoints>
            <Endpoint Name="ServiceEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>

</ServiceManifest>
```

Par ailleurs, le nom du point de terminaison doit être transmis au constructeur `HttpSysCommunicationListener` :

```csharp
 new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
 {
     return new WebHostBuilder()
         .UseHttpSys()
         .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
         .UseUrls(url)
         .Build();
 })
```

#### <a name="use-httpsys-with-a-static-port"></a>Utiliser HTTP.sys avec un port statique
Pour utiliser un port statique avec HTTP.sys, indiquez le numéro de port dans la configuration `Endpoint` :

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
    </Endpoints>
  </Resources>
```

#### <a name="use-httpsys-with-a-dynamic-port"></a>Utiliser HTTP.sys avec un port dynamique
Pour utiliser un port affecté dynamiquement avec HTTP.sys, omettez la propriété `Port` dans la configuration `Endpoint` :

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" />
    </Endpoints>
  </Resources>
```

Un port dynamique alloué par une configuration `Endpoint` fournit un seul port *par processus hôte*. Le modèle d’hébergement Service Fabric actuel permet à plusieurs instances de service et/ou réplicas d’être hébergés dans le même processus. Cela signifie que chacun d’eux partagera le même port lors de l’allocation par le biais de la configuration `Endpoint`. Plusieurs instances **HTTP.sys** peuvent partager un port à l’aide de la fonctionnalité de partage de port **HTTP.sys** sous-jacente. Mais elle n’est pas prise en charge par `HttpSysCommunicationListener` en raison des complications qu’elle induit pour les requêtes des clients. Pour une utilisation de port dynamique, il est suggéré d’utiliser Kestrel comme serveur web.

## <a name="kestrel-in-reliable-services"></a>Kestrel dans Reliable Services
Vous pouvez utiliser Kestrel dans un service Reliable Service en important le paquet NuGet **Microsoft.ServiceFabric.AspNetCore.Kestrel**. Ce package contient `KestrelCommunicationListener`, une implémentation de `ICommunicationListener`. `KestrelCommunicationListener` vous permet de créer un WebHost ASP.NET Core dans un service Reliable Services utilisant Kestrel comme serveur web.

Kestrel est un serveur Web multiplateforme pour ASP.NET Core. Contrairement à HTTP.sys, Kestrel n’utilise pas de gestionnaire de points de terminaison centralisé. Et également contrairement à HTTP.sys, Kestrel ne prend pas en charge le partage de port entre plusieurs processus. Chaque instance de Kestrel doit utiliser un port unique. Pour plus d’informations sur Kestrel, consultez les [Informations d’implémentation](/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-2.2).

![Diagramme de kestrel][4]

### <a name="kestrel-in-a-stateless-service"></a>Kestrel dans un service sans état
Pour utiliser `Kestrel` dans un service sans état, remplacez la méthode `CreateServiceInstanceListeners` et retournez une instance `KestrelCommunicationListener` :

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext =>
            new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                new WebHostBuilder()
                    .UseKestrel()
                    .ConfigureServices(
                        services => services
                            .AddSingleton<StatelessServiceContext>(serviceContext))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build();
            ))
    };
}
```

### <a name="kestrel-in-a-stateful-service"></a>Kestrel dans un service avec état
Pour utiliser `Kestrel` dans un service avec état, remplacez la méthode `CreateServiceReplicaListeners` et retournez une instance `KestrelCommunicationListener` :

```csharp
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new ServiceReplicaListener[]
    {
        new ServiceReplicaListener(serviceContext =>
            new KestrelCommunicationListener(serviceContext, (url, listener) =>
                new WebHostBuilder()
                    .UseKestrel()
                    .ConfigureServices(
                         services => services
                             .AddSingleton<StatefulServiceContext>(serviceContext)
                             .AddSingleton<IReliableStateManager>(this.StateManager))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build();
            ))
    };
}
```

Dans cet exemple, une instance singleton de `IReliableStateManager` est fournie au conteneur d’injection de dépendance WebHost. Cela n’est pas rigoureusement nécessaire, mais elle vous permet d’utiliser `IReliableStateManager` et Reliable Collections dans vos méthodes d’action de contrôleur MVC.

*Aucun* nom de configuration `Endpoint` n’est fourni à `KestrelCommunicationListener` dans un service avec état. La section suivante contient une explication plus détaillée à ce sujet.

### <a name="configure-kestrel-to-use-https"></a>Configurer Kestrel pour l’utilisation de HTTPS
Quand vous activez HTTPS avec Kestrel dans votre service, vous devez définir plusieurs options d’écoute. Mettez à jour le `ServiceInstanceListener` pour qu’il utilise un point de terminaison *EndpointHttps* et écoute sur un port spécifique (comme le port 443). Lors de la configuration de l’hôte web pour l’utilisation du serveur web Kestrel, vous devez configurer Kestrel pour qu’il écoute les adresses IPv6 sur toutes les interfaces réseau : 

```csharp
new ServiceInstanceListener(
serviceContext =>
    new KestrelCommunicationListener(
        serviceContext,
        "EndpointHttps",
        (url, listener) =>
        {
            ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

            return new WebHostBuilder()
                .UseKestrel(opt =>
                {
                    int port = serviceContext.CodePackageActivationContext.GetEndpoint("EndpointHttps").Port;
                    opt.Listen(IPAddress.IPv6Any, port, listenOptions =>
                    {
                        listenOptions.UseHttps(GetCertificateFromStore());
                        listenOptions.NoDelay = true;
                    });
                })
                .ConfigureAppConfiguration((builderContext, config) =>
                {
                    config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
                })

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
```

Pour obtenir un exemple complet dans un didacticiel, consultez [Configurer Kestrel pour l’utilisation de HTTPS](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#configure-kestrel-to-use-https).


### <a name="endpoint-configuration"></a>Configuration du point de terminaison
Aucune configuration `Endpoint` n’est requise pour utiliser Kestrel. 

Kestrel est un serveur web autonome simple. Contrairement à HTTP.sys (ou HttpListener), il n’a pas besoin d’une configuration `Endpoint` dans le fichier ServiceManifest.xml, car il ne nécessite pas l’inscription d’URL avant de commencer. 

#### <a name="use-kestrel-with-a-static-port"></a>Utiliser Kestrel avec un port statique
Vous pouvez configurer un port statique dans la configuration `Endpoint` du fichier ServiceManifest.xml pour une utilisation avec Kestrel. Même si cela n’est pas rigoureusement nécessaire, cela confère deux avantages potentiels :
 - Si le port n’est pas compris dans la plage de ports d’application, il est ouvert via le pare-feu du système d’exploitation par Service Fabric.
 - L’URL qui vous est fournie via `KestrelCommunicationListener` utilise ce port.

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
    </Endpoints>
  </Resources>
```

Si un `Endpoint` est configuré, son nom doit être transmis au constructeur `KestrelCommunicationListener` : 

```csharp
new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) => ...
```

Si le fichier ServiceManifest.xml n’utilise pas une configuration `Endpoint`, omettez le nom dans le constructeur `KestrelCommunicationListener`. Dans ce cas, un port dynamique est utilisé. Pour en savoir plus, consultez la section suivante à ce propos.

#### <a name="use-kestrel-with-a-dynamic-port"></a>Utiliser Kestrel avec un port dynamique
Kestrel ne peut pas utiliser l’attribution de port automatique à partir de la configuration `Endpoint` dans le fichier ServiceManifest.xml. En effet, l’affectation de port automatique d’une configuration `Endpoint` affecte un port unique par *processus hôte*, et un processus hôte unique peut contenir plusieurs instances de Kestrel. Cela ne fonctionne pas avec Kestrel, car il ne prend pas en charge le partage de port. Par conséquent, chaque instance de Kestrel doit être ouvert sur un port unique.

Pour utiliser l’affectation de port dynamique avec Kestrel, omettez entièrement la configuration `Endpoint` indiquée dans le fichier ServiceManifest.xml, et ne transmettez pas le nom d’un point de terminaison au constructeur `KestrelCommunicationListener`, comme suit :

```csharp
new KestrelCommunicationListener(serviceContext, (url, listener) => ...
```

Dans cette configuration, `KestrelCommunicationListener` sélectionne automatiquement un port inutilisé dans la plage de ports d’application.

Pour le protocole HTTPs, le point de terminaison doit être configuré avec le protocole HTTPs sans port spécifié dans le fichier ServiceManifest.xml, et transmettre le nom du point de terminaison au constructeur KestrelCommunicationListener.


## <a name="service-fabric-configuration-provider"></a>Fournisseur de configuration Service Fabric
La configuration d’application dans ASP.NET Core est basée sur des paires clé-valeur établies par le fournisseur de configuration. Lisez [Configuration dans ASP.NET Core](/aspnet/core/fundamentals/configuration/) pour en savoir plus sur le support de la configuration ASP.NET Core générale.

Cette section décrit comment le fournisseur de configuration Service Fabric s’intègre avec la configuration d’ASP.NET Core en important le package NuGet `Microsoft.ServiceFabric.AspNetCore.Configuration`.

### <a name="addservicefabricconfiguration-startup-extensions"></a>Extensions de démarrage AddServiceFabricConfiguration
Après avoir importé le package NuGet `Microsoft.ServiceFabric.AspNetCore.Configuration`, vous devez inscrire la source de configuration de Service Fabric avec l’API de configuration ASP.NET Core. Pour cela, recherchez les extensions **AddServiceFabricConfiguration** dans l’espace de noms `Microsoft.ServiceFabric.AspNetCore.Configuration` par rapport à `IConfigurationBuilder`.

```csharp
using Microsoft.ServiceFabric.AspNetCore.Configuration;

public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
        .AddServiceFabricConfiguration() // Add Service Fabric configuration settings.
        .AddEnvironmentVariables();
    Configuration = builder.Build();
}

public IConfigurationRoot Configuration { get; }
```

Désormais le service ASP.NET Core peut accéder aux paramètres de configuration de Service Fabric, comme tous les autres paramètres de l’application. Par exemple, vous pouvez utiliser le modèle d’options pour charger les paramètres dans des objets fortement typés.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<MyOptions>(Configuration);  // Strongly typed configuration object.
    services.AddMvc();
}
```
### <a name="default-key-mapping"></a>Mappage de clé par défaut
Par défaut, le fournisseur de configuration de Service Fabric inclut le nom du package, le nom de la section et le nom de la propriété. Ensemble, ils forment la clé de configuration ASP.NET Core, comme suit :
```csharp
$"{this.PackageName}{ConfigurationPath.KeyDelimiter}{section.Name}{ConfigurationPath.KeyDelimiter}{property.Name}"
```

Par exemple, si vous disposez d’un package de configuration nommé `MyConfigPackage` avec le contenu suivant, la valeur de configuration sera disponible sur ASP.NET Core `IConfiguration` via *MyConfigPackage:MyConfigSection:MyParameter*.
```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">  
  <Section Name="MyConfigSection">
    <Parameter Name="MyParameter" Value="Value1" />
  </Section>  
</Settings>
```
### <a name="service-fabric-configuration-options"></a>Options de configuration Service Fabric
Le fournisseur de configuration Service Fabric prend également en charge `ServiceFabricConfigurationOptions` pour modifier le comportement par défaut du mappage de clé.

#### <a name="encrypted-settings"></a>Paramètres chiffrés
Service Fabric prend en charge les paramètres chiffrés, comme le fournisseur de configuration Service Fabric. Les paramètres chiffrés ne sont pas déchiffrés par ASP.NET Core `IConfiguration` par défaut. Les valeurs chiffrées sont stockées là à la place. Mais si vous souhaitez déchiffrer la valeur à stocker dans ASP.NET Core IConfiguration, vous pouvez définir l’indicateur *DecryptValue* sur false dans l’extension `AddServiceFabricConfiguration`, comme suit :

```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    var builder = new ConfigurationBuilder()        
        .AddServiceFabricConfiguration(activationContext, (options) => options.DecryptValue = false); // set flag to decrypt the value
    Configuration = builder.Build();
}
```
#### <a name="multiple-configuration-packages"></a>Plusieurs packages de configuration
Service Fabric prend en charge plusieurs packages de configuration. Par défaut, le nom du package est inclus dans la clé de configuration. Mais vous pouvez définir l’indicateur `IncludePackageName` sur false, comme suit :
```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    var builder = new ConfigurationBuilder()        
        // exclude package name from key.
        .AddServiceFabricConfiguration(activationContext, (options) => options.IncludePackageName = false); 
    Configuration = builder.Build();
}
```
#### <a name="custom-key-mapping-value-extraction-and-data-population"></a>Mappage de clés personnalisé, extraction de valeur et remplissage des données
Le fournisseur de configuration Service Fabric prend également en charge des scénarios plus avancés pour personnaliser le mappage de clés avec `ExtractKeyFunc` et l’extraction personnalisée des valeurs avec `ExtractValueFunc`. Vous pouvez même modifier l’ensemble du processus de remplissage des données à partir de la configuration de Service Fabric jusqu’à la configuration d’ASP.NET Core à l’aide de `ConfigAction`.

Les exemples suivants illustrent comment utiliser `ConfigAction` pour personnaliser le remplissage des données :
```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    
    this.valueCount = 0;
    this.sectionCount = 0;
    var builder = new ConfigurationBuilder();
    builder.AddServiceFabricConfiguration(activationContext, (options) =>
        {
            options.ConfigAction = (package, configData) =>
            {
                ILogger logger = new ConsoleLogger("Test", null, false);
                logger.LogInformation($"Config Update for package {package.Path} started");

                foreach (var section in package.Settings.Sections)
                {
                    this.sectionCount++;

                    foreach (var param in section.Parameters)
                    {
                        configData[options.ExtractKeyFunc(section, param)] = options.ExtractValueFunc(section, param);
                        this.valueCount++;
                    }
                }

                logger.LogInformation($"Config Update for package {package.Path} finished");
            };
        });
  Configuration = builder.Build();
}
```

### <a name="configuration-updates"></a>Mises à jour de configuration
Le fournisseur de configuration Service Fabric prend également en charge les mises à jour de configuration. Vous pouvez utiliser ASP.NET Core `IOptionsMonitor` pour recevoir des notifications de modification, puis utiliser `IOptionsSnapshot` pour recharger les données de configuration. Pour plus d’informations, consultez [Options ASP.NET Core](/aspnet/core/fundamentals/configuration/options).

Ces options sont prises en charge par défaut. Aucune autre codage n’est nécessaire pour activer les mises à jour de configuration.

## <a name="scenarios-and-configurations"></a>Scénarios et configurations
Cette section fournit la combinaison de serveur web, configuration de port, options d’intégration de Service Fabric et divers paramètres que nous recommandons pour résoudre les scénarios suivants :
 - Services sans état d’ASP.NET Core exposés en externe
 - Services sans état d’ASP.NET Core interne uniquement
 - Services avec état d’ASP.NET Core interne uniquement

Un service **exposé en externe** expose un point de terminaison appelé depuis l’extérieur du cluster, généralement par le biais d’un équilibreur de charge.

Un service **interne uniquement** possède un point de terminaison qui est uniquement appelé à partir de l’intérieur du cluster.

> [!NOTE]
> En règle générale, les points de terminaison de service avec état ne doivent pas être exposés à Internet. Les clusters derrière des équilibreurs de charge qui n’ont pas conscients de la résolution de service Service Fabric, tels que Azure Load Balancer, ne pourront pas exposer les services avec état. En effet, l’équilibreur de charge n’est pas en mesure de localiser et d’acheminer le trafic vers le réplica approprié de service avec état . 

### <a name="externally-exposed-aspnet-core-stateless-services"></a>Services sans état d’ASP.NET Core exposés en externe
Kestrel est le serveur web suggéré pour les services frontend qui exposent des points de terminaison HTTP externes accessibles sur Internet. Sur Windows, HTTP.sys peut fournir des fonctionnalités de partage de port, ce qui vous permet d’héberger plusieurs services web sur le même ensemble de nœuds à l’aide du même port. Dans ce scénario, les services web sont différenciés par nom d’hôte ou par chemin d’accès, sans se baser sur un proxy ou une passerelle frontal pour assurer le routage HTTP.
 
Lorsqu’il est exposé à Internet, un service sans état doit utiliser un point de terminaison connu et stable accessible via un équilibreur de charge. Vous allez fournir cet URL aux utilisateurs de votre application. Nous vous recommandons la configuration suivante :

| Type | Recommandation | Notes |
| ---- | -------------- | ----- |
| Serveur web | Kestrel | Kestrel est le serveur web par défaut, car il est pris en charge sur Windows et Linux. |
| Configuration du port | statique | Un port statique connu doit être défini dans la configuration `Endpoints` du fichier ServiceManifest.xml, par exemple 80 pour HTTP et 443 pour HTTPS. |
| ServiceFabricIntegrationOptions | None | Utilisez l’option `ServiceFabricIntegrationOptions.None` lors de la configuration de l’intergiciel (middleware) d’intégration à Service Fabric afin que le service n’essaie pas de valider les demandes entrantes pour un identificateur unique. Les utilisateurs externes de votre application ne connaissent pas les informations d’identification uniques utilisées par l’intergiciel (middleware). |
| Nombre d'instances | -1 | Dans les scénarios d’utilisation classiques, le paramètre du nombre d’instances doit être défini sur *-1*. Cela est fait de sorte qu’une instance soit disponible sur tous les nœuds recevant le trafic provenant d’un équilibreur de charge. |

Si plusieurs services exposés en externe partagent le même ensemble de nœuds, vous pouvez utiliser HTTP.sys avec un chemin URL unique, mais stable. Vous pouvez accomplir cela en modifiant l’URL fournie lors de la configuration d’IWebHost. Notez que cela s’applique uniquement à HTTP.sys.

 ```csharp
 new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
 {
     url += "/MyUniqueServicePath";
 
     return new WebHostBuilder()
         .UseHttpSys()
         ...
         .UseUrls(url)
         .Build();
 })
 ```

### <a name="internal-only-stateless-aspnet-core-service"></a>Service d’ASP.NET Core sans état interne uniquement
Les services sans état qui sont appelés uniquement à partir du cluster doivent utiliser des URL uniques et des ports affectés dynamiquement afin d’assurer une coopération entre plusieurs services. Nous vous recommandons la configuration suivante :

| Type | Recommandation | Notes |
| ---- | -------------- | ----- |
| Serveur web | Kestrel | Même si vous pouvez utiliser HTTP.sys pour les services sans état internes, Kestrel est le meilleur serveur pour permettre à plusieurs instances de service de partager un hôte.  |
| Configuration du port | affecté de manière dynamique | Plusieurs réplicas d’un service avec état peuvent partager un processus hôte ou un système d’exploitation hôte, et nécessitent donc des ports uniques. |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | Avec l’affectation de port dynamique, ce paramètre empêche le problème d’erreur d’identité décrit précédemment. |
| InstanceCount | n'importe laquelle | Le paramètre du nombre d’instances peut être défini sur toute valeur nécessaire au bon fonctionnement du service. |

### <a name="internal-only-stateful-aspnet-core-service"></a>Service d’ASP.NET Core avec état interne uniquement
Les services avec état qui sont appelés uniquement à partir du cluster doivent utiliser des ports affectés dynamiquement afin d’assurer une coopération entre plusieurs services. Nous vous recommandons la configuration suivante :

| Type | Recommandation | Notes |
| ---- | -------------- | ----- |
| Serveur web | Kestrel | Le `HttpSysCommunicationListener` n’est pas conçu pour être utilisé par les services avec état dans lesquels les réplicas partagent un processus hôte. |
| Configuration du port | affecté de manière dynamique | Plusieurs réplicas d’un service avec état peuvent partager un processus hôte ou un système d’exploitation hôte, et nécessitent donc des ports uniques. |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | Avec l’affectation de port dynamique, ce paramètre empêche le problème d’erreur d’identité décrit précédemment. |

## <a name="next-steps"></a>Étapes suivantes
[Débogage de votre application Service Fabric à l’aide de Visual Studio](service-fabric-debugging-your-application.md)


<!--Image references-->
[0]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-standalone.png
[1]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-servicefabric.png
[2]:./media/service-fabric-reliable-services-communication-aspnetcore/integration.png
[3]:./media/service-fabric-reliable-services-communication-aspnetcore/httpsys.png
[4]:./media/service-fabric-reliable-services-communication-aspnetcore/kestrel.png
