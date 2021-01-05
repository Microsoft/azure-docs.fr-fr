---
title: Se connecter aux services et communiquer avec eux dans Azure Service Fabric
description: Découvrez comment résoudre, vous connecter aux services et communiquer avec ces derniers dans Service Fabric.
ms.topic: conceptual
ms.date: 11/01/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 11f525eba89dc963deee0ba9a86566361ef644de
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576296"
---
# <a name="connect-and-communicate-with-services-in-service-fabric"></a>Se connecter aux services et communiquer avec eux dans Service Fabric.
Dans Service Fabric, un service s’exécute quelque part dans un cluster Service Fabric, généralement réparti sur plusieurs machines virtuelles. Il peut être déplacé d’un endroit à un autre, soit par le propriétaire du service, soit automatiquement par Service Fabric. Les services ne sont pas liés statiquement à une machine ou une adresse particulière.

Une application Service Fabric est généralement composée de nombreux services différents qui effectuent chacun une tâche spécialisée. Ces services peuvent communiquer entre eux pour former une fonction complète, telle que le rendu des différentes parties d’une application web. Il existe également des applications clientes qui se connectent aux services et communiquent avec eux. Ce document décrit comment configurer la communication avec et entre vos services dans Service Fabric.

## <a name="bring-your-own-protocol"></a>Apportez votre propre protocole
Service Fabric permet de gérer le cycle de vie de vos services, mais ne décide pas des rôles de ces derniers. Cela inclut la communication. L’ouverture de votre service par Service Fabric permet au service de configurer un point de terminaison pour les demandes entrantes à l’aide du protocole ou de la pile de communication du service de votre choix. Votre service écoute sur une adresse **IP:port** normale à l’aide d’un schéma d’adressage quelconque, tel qu’un URI. Plusieurs instances de service ou réplicas peuvent partager un processus hôte, auquel cas ils devront utiliser des ports différents ou un mécanisme de partage de ports, tel que le pilote du noyau http.sys dans Windows. Dans les deux cas, chaque instance de service ou réplica dans un processus hôte doit être adressable de manière unique.

![points de terminaison de service][1]

## <a name="service-discovery-and-resolution"></a>Détection et résolution de service
Dans un système distribué, le service que vous exécutez peut se déplacer d’un ordinateur vers un autre au fil du temps. Cela peut se produire pour diverses raisons, y compris l’équilibrage des ressources, les mises à niveau, les basculements ou l’augmentation des tailles des instances. Cela signifie que les adresses de point de terminaison de service sont modifiées lorsque le service se déplace vers des nœuds dont les adresses IP sont différentes ; celles-ci peuvent s’ouvrir sur des ports différents si le service utilise un port sélectionné de manière dynamique.

![Répartition des services][7]

Service Fabric fournit un service de détection et de résolution appelé Service d’attribution de noms. Le Service d’attribution de noms tient une table qui mappe les instances de service nommées sur les adresses de point de terminaison qu’elles écoutent. Toutes les instances de service nommées dans Service Fabric ont des noms uniques représentés par les URI. Par exemple : `"fabric:/MyApplication/MyService"`. Le nom du service ne change pas au cours de la durée de vie du service ; seules les adresses de point de terminaison peuvent changer lorsque les services sont déplacés. C’est le même principe que pour les sites web, qui ont des URL constantes, mais dont l’adresse IP peut changer. Et comme les DNS sur le web, qui résolvent les URL des sites web en adresses IP, Service Fabric dispose d’un serveur d’inscriptions avancé qui mappe les noms de service sur leur adresse de point de terminaison.

![Diagramme qui montre que Service Fabric a un bureau d’enregistrement qui mappe les noms de service à leur adresse de point de terminaison.][2]

La résolution et la connexion aux services impliquent l’exécution des étapes suivantes en boucle :

* **Résoudre** : Obtenir le point de terminaison publié par un service à partir du Service d’attribution de noms.
* **Connecter** : Se connecter au service à l’aide du protocole que ce dernier utilise sur le point de terminaison en question.
* **Réessayer** : Une tentative de connexion peut échouer pour diverses raisons. Par exemple : si le service a été déplacé depuis la dernière fois que l’adresse de point de terminaison a été résolue. Si la tentative de connexion échoue, les étapes précédentes de résolution et de connexion doivent être réessayées, et ce cycle se répète jusqu’à ce que la connexion aboutisse.

## <a name="connecting-to-other-services"></a>Connexion à d’autres services
Les services se connectant entre eux, en général à l’intérieur d’un cluster, peuvent accéder directement aux points de terminaison d’autres services, car les nœuds d’un cluster se trouvent sur le même réseau local. Afin de faciliter la connexion entre les services, Service Fabric fournit des services supplémentaires qui utilisent le service d’affectation de noms : un service DNS et un service de proxy inverse.


### <a name="dns-service"></a>Service DNS
Dans la mesure où de nombreux services, en particulier les services en conteneur, peuvent avoir un nom d’URL existant, il est très pratique de pouvoir utiliser le protocole DNS standard (plutôt que le protocole d’affectation de noms), notamment dans les scénarios impliquant le développement et le transfert d’applications. C’est là précisément qu’intervient le service DNS. Il vous permet de mapper les noms DNS à un nom de service et, par conséquent, de résoudre les adresses IP des points de terminaison. 

Comme le montre l’illustration suivante, le service DNS (exécuté dans le cluster Service Fabric), mappe les noms DNS aux noms de service qui sont ensuite résolus par le service d’affectation de noms pour renvoyer les adresses de point de terminaison auxquelles se connecter. Le nom DNS du service est fourni au moment de la création. 

![Diagramme qui montre comment le service DNS (exécuté dans le cluster Service Fabric), mappe les noms DNS aux noms de service qui sont ensuite résolus par le service d’affectation de noms pour renvoyer les adresses de point de terminaison auxquelles se connecter.][9]

Pour plus d’informations sur l’utilisation du service DNS, consultez l’article [Service DNS dans Azure Service Fabric](service-fabric-dnsservice.md).

### <a name="reverse-proxy-service"></a>Service de proxy inverse
Le proxy inverse traite les services dans le cluster qui expose des points de terminaison HTTP, y compris HTTPS. Le proxy inverse simplifie l’appel d’autres services et de leurs méthodes grâce à un format URI spécifique, et gère les étapes de résolution, de connexion et de reconnexion indispensables pour permettre à un service de communiquer avec un autre à l’aide du service de nommage. En d’autres termes, il masque le service d’affectation de noms lors de l’appel d’autres services de sorte que la procédure est aussi simple que l’appel d’une URL.

![Diagramme qui montre comment le proxy inverse traite les services dans le cluster qui expose des points de terminaison HTTP, y compris HTTPS.][10]

Pour plus d’informations sur l’utilisation du service de proxy inverse, consultez l’article [Proxy inverse dans Azure Service Fabric](service-fabric-reverseproxy.md).

## <a name="connections-from-external-clients"></a>Connexions des clients externes
Les services se connectant entre eux, en général à l’intérieur d’un cluster, peuvent accéder directement aux points de terminaison d’autres services, car les nœuds d’un cluster se trouvent sur le même réseau local. Cependant, dans certains environnements, un cluster peut se trouver derrière un équilibrage de charge. Celui-ci achemine le trafic d’entrée à travers un ensemble limité de ports. Dans ces cas, les services peuvent toujours communiquer entre eux et résoudre les adresses à l’aide du Service d’attribution de noms. Cependant, des étapes supplémentaires doivent être suivies afin d’autoriser les clients externes à se connecter aux services.

## <a name="service-fabric-in-azure"></a>Service Fabric dans Azure
Un cluster Service Fabric dans Azure est placé derrière un équilibrage de charge Azure. Tout trafic externe vers le cluster doit passer par l’équilibrage de charge. L’équilibrage de charge transmet automatiquement le trafic entrant sur un port donné vers un *nœud* aléatoire ayant le même port ouvert. L’Azure Load Balancer (équilibrage de charge Azure) ne connaît que les ports ouverts sur les *nœuds*. Il ne connaît pas les ports ouverts par des *services* individuels.

![Topologie d’équilibrage de charge Azure et de Service Fabric][3]

Par exemple : les éléments suivants doivent être configurés pour accepter le trafic externe sur le port **80**:

1. Écrivez un service qui écoute sur le port 80. Configurez le port 80 dans le fichier ServiceManifest.xml du service et ouvrez un écouteur dans le service, tel qu’un serveur web auto-hébergé.

    ```xml
    <Resources>
        <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>
    ```
    ```csharp
        class HttpCommunicationListener : ICommunicationListener
        {
            ...

            public Task<string> OpenAsync(CancellationToken cancellationToken)
            {
                EndpointResourceDescription endpoint =
                    serviceContext.CodePackageActivationContext.GetEndpoint("WebEndpoint");

                string uriPrefix = $"{endpoint.Protocol}://+:{endpoint.Port}/myapp/";

                this.httpListener = new HttpListener();
                this.httpListener.Prefixes.Add(uriPrefix);
                this.httpListener.Start();

                string publishUri = uriPrefix.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);
                return Task.FromResult(publishUri);
            }

            ...
        }

        class WebService : StatelessService
        {
            ...

            protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
            {
                return new[] { new ServiceInstanceListener(context => new HttpCommunicationListener(context))};
            }

            ...
        }
    ```
    ```java
        class HttpCommunicationlistener implements CommunicationListener {
            ...

            @Override
            public CompletableFuture<String> openAsync(CancellationToken arg0) {
                EndpointResourceDescription endpoint =
                    this.serviceContext.getCodePackageActivationContext().getEndpoint("WebEndpoint");
                try {
                    HttpServer server = com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(endpoint.getPort()), 0);
                    server.start();

                    String publishUri = String.format("http://%s:%d/",
                        this.serviceContext.getNodeContext().getIpAddressOrFQDN(), endpoint.getPort());
                    return CompletableFuture.completedFuture(publishUri);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }

            ...
        }

        class WebService extends StatelessService {
            ...

            @Override
            protected List<ServiceInstanceListener> createServiceInstanceListeners() {
                <ServiceInstanceListener> listeners = new ArrayList<ServiceInstanceListener>();
                listeners.add(new ServiceInstanceListener((context) -> new HttpCommunicationlistener(context)));
                return listeners;       
            }

            ...
        }
    ```
2. Créez un cluster Service Fabric dans Azure et spécifiez le port **80** comme port de point de terminaison personnalisé pour le type de nœud qui héberge le service. Si vous avez plusieurs types de nœud, vous pouvez définir une *contrainte de placement* sur le service pour vous assurer qu’il s’exécute uniquement sur le type de nœud dont le port du point de terminaison personnalisé est ouvert.

    ![Ouvrir un port sur un type de nœud][4]
3. Une fois que le cluster a été créé, configurez l’équilibrage de charge Azure dans le groupe de ressources du cluster pour transférer le trafic sur le port 80. Lorsque vous créez un cluster via le portail Azure, il est configuré automatiquement pour chaque port de point de terminaison personnalisé ayant été configuré.

    ![Capture d’écran qui met en surbrillance le champ port principal sous les règles d’équilibrage de charge.][5]
4. L’équilibrage de charge Azure utilise une sonde pour déterminer s’il faut envoyer le trafic vers un nœud particulier. La sonde vérifie régulièrement un point de terminaison sur chaque nœud pour déterminer si le nœud répond. Si la sonde ne reçoit aucune réponse après un nombre de fois prédéterminé, l’équilibrage de charge n’envoie plus de trafic vers ce nœud. Lorsque vous créez un cluster via le portail Azure, une sonde est configurée automatiquement pour chaque port de point de terminaison personnalisé ayant été configuré.

    ![Transférer le trafic dans l’équilibrage de charge Azure][8]

Gardez à l’esprit que l’Azure Load Balancer (équilibrage de charge Azure) et la sonde ont uniquement connaissance des *nœuds*, et non des *services* s’exécutant sur les nœuds. L’équilibrage de charge Azure envoie toujours le trafic aux nœuds qui répondent à la sonde. Vous devez donc veiller à ce que les services soient disponibles sur les nœuds en mesure de répondre à cette dernière.

## <a name="reliable-services-built-in-communication-api-options"></a>Reliable Services : Options d’API de communication intégrées
L’infrastructure Reliable Services est livrée avec plusieurs options de communication intégrées. Le choix de l’API dépend du modèle de programmation utilisé, de la structure de communication et du langage de programmation dans lequel vos services sont écrits.

* **Aucun protocole spécifique** :  Si vous pouvez choisir n’importe quelle infrastructure de communication, mais que vous souhaitez une solution rapidement opérationnelle, l’option idéale est le [service à distance](service-fabric-reliable-services-communication-remoting.md), qui rend possible des appels de procédure distante fortement typés pour Reliable Services et Reliable Actors. Il s'agit de la façon la plus simple et la plus rapide de prendre en main la communication avec les services. Le service à distance gère la résolution des adresses de service ainsi que la connexion, les nouvelles tentatives et la gestion des erreurs. Il est disponible pour les applications C# et Java.
* **HTTP** : Pour la communication sans langage spécifié, HTTP fournit un choix normalisé avec des outils et des serveurs HTTP disponibles dans plusieurs langues, le tout pris en charge par Service Fabric. Les services peuvent utiliser n’importe quelle pile HTTP, notamment l[’API web ASP.NET](./service-fabric-reliable-services-communication-aspnetcore.md) pour les applications C#. Les clients écrits en C# peuvent exploiter les classes `ICommunicationClient` et `ServicePartitionClient`. En revanche, pour ceux écrits en Java, utilisez les classes `CommunicationClient` et `FabricServicePartitionClient`[pour la résolution de service, les connexions HTTP et les boucles de nouvelle tentative](service-fabric-reliable-services-communication.md).
* **WCF** : Si votre code existant utilise WCF comme infrastructure de communication, vous pouvez utiliser `WcfCommunicationListener` pour le côté serveur et les classes `WcfCommunicationClient` et `ServicePartitionClient` pour le client. Cela n’est possible que pour les applications C# sur des clusters basés sur Windows. Pour plus d’informations, consultez cet article sur l’ [implémentation WCF de la pile de communication](service-fabric-reliable-services-communication-wcf.md).

## <a name="using-custom-protocols-and-other-communication-frameworks"></a>Utilisation des protocoles personnalisés et d’autres infrastructures de communication
Les services peuvent utiliser un protocole ou une infrastructure quelconque pour la communication, qu’il s’agisse d’un protocole binaire personnalisé passant par des sockets TCP ou d’événements de diffusion passant par [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) ou [Azure IoT Hub](https://azure.microsoft.com/services/iot-hub/). Service Fabric fournit des API de communication sur lesquelles vous pouvez connecter votre pile de communication, tout en maintenant pour vous la transparence de la découverte et de la connexion. Pour plus d’informations, consultez cet article sur le [modèle de communication de service fiable](service-fabric-reliable-services-communication.md) .

## <a name="next-steps"></a>Étapes suivantes
Découvrez-en plus sur les concepts et les API disponibles dans le [modèle de communication des Reliable Services](service-fabric-reliable-services-communication.md), puis prenez rapidement en main le [service distant](service-fabric-reliable-services-communication-remoting.md), ou allez plus en profondeur pour découvrir comment écrire un écouteur de communication à l’aide de [l’API web avec auto-hébergement OWIN](./service-fabric-reliable-services-communication-aspnetcore.md).

[1]: ./media/service-fabric-connect-and-communicate-with-services/serviceendpoints.png
[2]: ./media/service-fabric-connect-and-communicate-with-services/namingservice.png
[3]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancertopology.png
[4]: ./media/service-fabric-connect-and-communicate-with-services/nodeport.png
[5]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerport.png
[7]: ./media/service-fabric-connect-and-communicate-with-services/distributedservices.png
[8]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerprobe.png
[9]: ./media/service-fabric-connect-and-communicate-with-services/dns.png
[10]: ./media/service-fabric-reverseproxy/internal-communication.png
