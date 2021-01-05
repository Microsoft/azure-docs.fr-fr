---
title: Communication à distance des services à l’aide de Java dans Azure Service Fabric
description: La communication à distance dans Service Fabric permet aux clients et aux services de communiquer avec les services Java en utilisant un appel de procédure distante.
author: PavanKunapareddyMSFT
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-java
ms.author: pakunapa
ms.openlocfilehash: d53d20510db70d81aab796efab48de40c880bb3a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87316122"
---
# <a name="service-remoting-in-java-with-reliable-services"></a>Communication à distance des services dans Java avec Reliable Services
> [!div class="op_single_selector"]
> * [C# sur Windows](service-fabric-reliable-services-communication-remoting.md)
> * [Java sur Linux](service-fabric-reliable-services-communication-remoting-java.md)
>
>

Pour les services qui ne sont pas liés à une pile ou un protocole de communication particulier, comme WebAPI, Windows Communication Foundation (WCF) ou autres, l’infrastructure Reliable Services fournit un mécanisme de communication à distance pour configurer rapidement et facilement des appels de procédure distante pour les services.  Cet article explique comment configurer les appels de procédure distante pour les services écrits avec Java.

## <a name="set-up-remoting-on-a-service"></a>Configuration de la communication à distance sur un service
La configuration de communication à distance pour un service s'effectue en deux étapes simples :

1. Créez une interface pour implémenter votre service. Cette interface définit les méthodes disponibles pour un appel de procédure distante sur votre service. Ces méthodes doivent être des méthodes asynchrones retournant des tâches. L'interface doit implémenter `microsoft.serviceFabric.services.remoting.Service` pour signaler que le service dispose d'une interface de communication à distance.
2. Utilisez un écouteur de communication à distance dans votre service. Il s'agit d'une implémentation `CommunicationListener` qui fournit des fonctionnalités de communication à distance. `FabricTransportServiceRemotingListener` peut être utilisé pour créer un écouteur de communication à distance à l’aide du protocole de transport de communication à distance par défaut.

Par exemple, le service sans état suivant expose une méthode unique pour obtenir « Hello World » sur un appel de procédure distante :

```java
import java.util.ArrayList;
import java.util.concurrent.CompletableFuture;
import java.util.List;
import microsoft.servicefabric.services.communication.runtime.ServiceInstanceListener;
import microsoft.servicefabric.services.remoting.Service;
import microsoft.servicefabric.services.runtime.StatelessService;

public interface MyService extends Service {
    CompletableFuture<String> helloWorldAsync();
}

class MyServiceImpl extends StatelessService implements MyService {
    public MyServiceImpl(StatelessServiceContext context) {
       super(context);
    }

    public CompletableFuture<String> helloWorldAsync() {
        return CompletableFuture.completedFuture("Hello!");
    }

    @Override
    protected List<ServiceInstanceListener> createServiceInstanceListeners() {
        ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
        listeners.add(new ServiceInstanceListener((context) -> {
            return new FabricTransportServiceRemotingListener(context,this);
        }));
        return listeners;
    }
}
```

> [!NOTE]
> Les arguments et les types de retour dans l’interface du service peuvent être de type simple, complexe ou personnalisé, mais ils doivent être sérialisables.
>
>

## <a name="call-remote-service-methods"></a>Méthodes d’appel de service distant
L’appel de méthodes sur un service à l’aide de la pile de communication à distance est effectué avec un proxy local pour le service via la classe `microsoft.serviceFabric.services.remoting.client.ServiceProxyBase` . La méthode `ServiceProxyBase` crée un proxy local à l’aide de l’interface implémentée par le service. Avec ce proxy, vous pouvez appeler simplement et à distance des méthodes sur l'interface.

```java

MyService helloWorldClient = ServiceProxyBase.create(MyService.class, new URI("fabric:/MyApplication/MyHelloWorldService"));

CompletableFuture<String> message = helloWorldClient.helloWorldAsync();

```

L'infrastructure de communication à distance propage les exceptions levées au niveau du service au client. Par conséquent, la logique de gestion des exceptions au niveau du client à l’aide de `ServiceProxyBase` peut directement traiter les exceptions levées par le service.

## <a name="service-proxy-lifetime"></a>Durée de vie du proxy du service
Comme la création de proxy de service est une opération légère, vous pouvez en créer autant de fois que nécessaire. Les instances de proxy de service peuvent être réutilisées tant qu’elles sont nécessaires. Si un appel de procédure distante lève une exception, vous pouvez toujours réutiliser la même instance de proxy. Chaque proxy de service contient un client de communication qui permet d’envoyer des messages sur le réseau. Lors de l’invocation d’appels distants, des vérifications en interne sont effectuées pour déterminer si le client de communication est valide. Selon les résultats de ces vérifications, le client de communication est recréé, si nécessaire. Par conséquent, si une exception se produit, il est inutile de recréer `ServiceProxy`.

### <a name="serviceproxyfactory-lifetime"></a>Durée de vie de la fabrique ServiceProxyFactory
[FabricServiceProxyFactory](/java/api/microsoft.servicefabric.services.remoting.client.fabricserviceproxyfactory) est une fabrique qui crée un proxy pour différentes interfaces de communication à distance. Si vous utilisez l’API `ServiceProxyBase.create` pour la création du proxy, l’infrastructure crée ensuite un `FabricServiceProxyFactory`.
Il est utile d’en créer un manuellement lorsque vous devez remplacer les propriétés de [ServiceRemotingClientFactory](/java/api/microsoft.servicefabric.services.remoting.client.serviceremotingclientfactory).
La fabrique est une opération coûteuse. `FabricServiceProxyFactory` conserve le cache des clients de communication.
Il est recommandé de mettre en cache `FabricServiceProxyFactory` aussi longtemps que possible.

## <a name="remoting-exception-handling"></a>Gestion des exceptions à distance
Toutes les exceptions distantes levées par l’API du service sont renvoyées au client sous la forme RuntimeException ou FabricException.

Le proxy de service gère toutes les exceptions de basculement pour la partition de service pour laquelle il a été créé. Il résout à nouveau les points de terminaison s’il existe des exceptions de basculement (non temporaires) et retente l’appel avec le point de terminaison correct. Le nombre de tentatives pour l’exception de basculement est illimité.
En cas d’exceptions temporaires, il retente uniquement l’appel.

Les paramètres de nouvelle tentative par défaut sont fournis par [OperationRetrySettings](/java/api/microsoft.servicefabric.services.communication.client.operationretrysettings).
Vous pouvez configurer ces valeurs en passant l’objet OperationRetrySettings au constructeur ServiceProxyFactory.

## <a name="next-steps"></a>Étapes suivantes
* [Sécurisation des communications pour Reliable Services](service-fabric-reliable-services-secure-communication-java.md)
