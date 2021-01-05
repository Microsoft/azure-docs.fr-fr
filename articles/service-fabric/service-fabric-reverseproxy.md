---
title: Proxy inverse Azure Service Fabric
description: Utilisez le proxy inverse de Service Fabric pour assurer les communications avec les microservices de l’intérieur et de l’extérieur du cluster.
author: BharatNarasimman
ms.topic: conceptual
ms.date: 11/03/2017
ms.author: bharatn
ms.custom: devx-track-csharp
ms.openlocfilehash: fd8e6dd712801de49971c1ef27cea664d73a4cb0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005909"
---
# <a name="reverse-proxy-in-azure-service-fabric"></a>Proxy inverse dans Azure Service Fabric
Le proxy inverse intégré à Azure Service Fabric permet aux microservices exécutés dans un cluster Service Fabric de découvrir d’autres services détenant des points de terminaison http et de communiquer avec ces services.

## <a name="microservices-communication-model"></a>Modèle de communication des microservices
Dans Service Fabric, les microservices s’exécutent sur un sous-ensemble de nœuds au sein du cluster et peuvent migrer entre les nœuds, pour des raisons diverses. Ainsi, les points de terminaison associés aux microservices peuvent changer de manière dynamique. Pour découvrir d’autres services dans le cluster et communiquer avec ces derniers, un microservice doit passer par les étapes suivantes :

1. Résoudre l’emplacement du service avec le service de nommage
2. Connectez-vous au service.
3. Encapsuler les étapes précédentes dans une boucle qui implémente la résolution de service, puis réessayer des stratégies à appliquer en cas d’échec de la connexion

Pour plus d’informations, consultez la page [Se connecter et communiquer avec les services](service-fabric-connect-and-communicate-with-services.md).

### <a name="communicating-by-using-the-reverse-proxy"></a>Communiquer avec le proxy inverse
Le proxy inverse est un service qui s’exécute sur chaque nœud et qui gère la résolution des points de terminaison, les nouvelles tentatives automatiques et autres échecs de connexion pour le compte des services clients. Le proxy inverse peut être configuré pour appliquer différentes stratégies à mesure qu’il gère les demandes émanant des services clients. Utiliser un proxy inverse permet au service client de recourir à toutes les bibliothèques de communication HTTP côté client et ne nécessite aucune résolution spéciale ou logique de nouvelle tentative dans le service. 

Le proxy inverse expose un ou plusieurs points de terminaison sur le nœud local que les services clients peuvent utiliser pour envoyer des demandes à d’autres services.

![Communications internes][1]

> [!NOTE]
> **Plateformes prises en charge**
>
> Le proxy inverse dans Service Fabric prend en charge les plateformes suivantes :
> * *Cluster Windows* : Windows 8 et versions ultérieures, ou Windows Server 2012 et versions ultérieures
> * *Cluster Linux* : le proxy inverse n’est pas disponible pour les clusters Linux.
>

## <a name="reaching-microservices-from-outside-the-cluster"></a>Atteindre les microservices de l’extérieur du cluster
Le modèle de communication externe par défaut des microservices est un modèle d’adhésion, avec lequel aucun service n’est accessible directement à partir des clients externes. [Azure Load Balancer](../load-balancer/load-balancer-overview.md), qui représente une limite réseau située entre les microservices et les clients externes, effectue la traduction des adresses réseau et transfère les requêtes externes vers des points de terminaison IP:port internes. Pour que le point de terminaison d’un microservice soit accessible aux clients externes, vous devez tout d’abord configurer l’équilibreur de charge de sorte qu’il transfère le trafic vers chaque port utilisé par le service au sein du cluster. En outre, la plupart des microservices, en particulier les microservices avec état, ne résident pas sur tous les nœuds du cluster. Les microservices peuvent se déplacer entre les nœuds lors d’un basculement. Dans ce cas, l’équilibreur de charge ne peut pas déterminer efficacement l’emplacement du nœud cible des réplicas auxquels il doit transférer le trafic.

### <a name="reaching-microservices-via-the-reverse-proxy-from-outside-the-cluster"></a>Atteindre les microservices par l’intermédiaire du proxy inverse à partir de l’extérieur du cluster
Au lieu de configurer le port d’un service donné dans l’équilibreur de charge, vous pouvez configurer uniquement le port du proxy inverse dans l’équilibreur de charge. Cette configuration permet aux clients se trouvant en dehors du cluster d’atteindre les services à l’intérieur du cluster avec le proxy inverse sans qu’une configuration supplémentaire soit nécessaire.

![Communications externes][0]

> [!WARNING]
> Lorsque vous configurez le port du proxy inverse dans l’équilibreur de charge, les systèmes se trouvant à l’extérieur du cluster peuvent atteindre tous les microservices du cluster exposant un point de terminaison HTTP. Cela signifie que des microservices destinés à être internes pourraient se révéler détectables par un utilisateur malveillant déterminé. Il peut en résulter de graves vulnérabilités qui peuvent être exploitées, notamment dans les cas de figure suivants :
>
> * Un utilisateur malveillant lance une attaque par déni de service en appelant à de nombreuses reprises un service interne qui ne dispose pas d’une surface d’attaque suffisamment renforcée.
> * Un utilisateur malveillant remet des paquets incorrects à un service interne, ce qui entraîne un comportement inattendu.
> * Un service destiné à être interne retourne des informations privées ou sensibles censées être hors de portée des services extérieurs au cluster, ce qui les expose à un utilisateur malveillant. 
>
> Veillez à bien comprendre et à limiter les conséquences potentielles sur la sécurité de votre cluster et des applications associées avant de rendre le port du proxy inverse public. 
>


## <a name="uri-format-for-addressing-services-by-using-the-reverse-proxy"></a>Format d’URI pour l’adressage des services avec le proxy inverse
Le proxy inverse utilise un format d’URI (Uniform Resource Identifier) spécifique pour identifier la partition de service vers laquelle une requête entrante doit être transférée :

```
http(s)://<Cluster FQDN | internal IP>:Port/<ServiceInstanceName>/<Suffix path>?PartitionKey=<key>&PartitionKind=<partitionkind>&ListenerName=<listenerName>&TargetReplicaSelector=<targetReplicaSelector>&Timeout=<timeout_in_seconds>
```

* **http(s) :** le proxy inverse peut être configuré pour accepter le trafic HTTP ou HTTPS. Pour le transfert HTTPS, reportez-vous à [Se connecter à un service sécurisé avec le proxy inverse](service-fabric-reverseproxy-configure-secure-communication.md) une fois que vous avez configuré le proxy inverse pour écouter sur HTTPS.
* **Nom de domaine complet du cluster | IP interne :** pour les clients externes, vous pouvez configurer le proxy inverse de façon à le rendre accessible par le biais du domaine du cluster, par exemple mycluster.eastus.cloudapp.azure.com. Par défaut, le proxy inverse s’exécute sur chacun des nœuds. Pour le trafic interne, le proxy inverse est accessible sur l’hôte local ou sur n’importe quelle adresse IP de nœud interne, par exemple 10.0.0.1.
* **Port :** port (par exemple 19081) qui est spécifié pour le proxy inverse.
* **ServiceInstanceName :** nom complet de l’instance de service déployée que vous tentez d’atteindre sans le schéma « fabric:/ ». Par exemple, pour atteindre le service *fabric:/myapp/myservice/* , vous pouvez utiliser *myapp/myservice*.

    Le nom d’instance de service respecte la casse. L’utilisation d’une casse différente pour le nom d’instance de service dans l’URL entraîne l’échec des demandes avec l’erreur 404 (Introuvable).
* **Suffix path :** chemin de l’URL, par exemple *myapi/values/add/3*,qui est associée au service auquel vous souhaitez vous connecter.
* **PartitionKey :** pour un service partitionné, il s’agit de la clé calculée associée à la partition que vous souhaitez atteindre. Il ne s’agit *pas* du GUID d’ID de la partition. Ce paramètre n’est pas obligatoire pour les services qui utilisent le schéma de partition de singleton.
* **PartitionKind :** schéma de partition du service. Il peut s’agir de Named ou Int64Range. Ce paramètre n’est pas obligatoire pour les services qui utilisent le schéma de partition de singleton.
* **ListenerName :** les points de terminaison du service se présentent sous la forme {"Endpoints":{"Listener1":"Endpoint1","Listener2":"Endpoint2"...}}. Lorsque le service expose plusieurs points de terminaison, cela permet d’identifier le point de terminaison auquel la demande client doit être transférée. Ce paramètre peut être omis si le service n’a qu’un seul écouteur.
* **TargetReplicaSelector :** spécifie le mode de sélection à utiliser pour le réplica cible ou l’instance.
  * Lorsque le service cible est avec état, le paramètre TargetReplicaSelector peut être défini sur :  « PrimaryReplica », « RandomSecondaryReplica » ou « RandomReplica ». Lorsque ce paramètre n’est pas spécifié, la valeur par défaut est « PrimaryReplica ».
  * Lorsque le service cible est sans état, le proxy inverse transfère la demande à une instance aléatoire de la partition de service.
* **Timeout :**  spécifie le délai d’attente de la requête HTTP créée par le proxy inverse pour le service, pour le compte de la requête du client. La valeur par défaut est 120 secondes. Il s'agit d'un paramètre facultatif.

### <a name="example-usage"></a>Exemple d’utilisation
Prenons l’exemple du service *fabric:/MyApp/MyService*, qui ouvre un écouteur HTTP sur l’URL suivante :

```
http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/
```

Voici les ressources du service :

* `/index.html`
* `/api/users/<userId>`

Si le service utilise le schéma de partitionnement singleton, les paramètres de chaîne de requête *PartitionKey* et *PartitionKind* ne sont pas requis ; le service est accessible avec la passerelle de la façon suivante :

* En externe : `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService`
* En interne : `http://localhost:19081/MyApp/MyService`

Si le service utilise le schéma de partitionnement Int64 uniforme, les paramètres de chaîne de requête *PartitionKey* et *PartitionKind* doivent être utilisés pour atteindre une partition du service :

* En externe : `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`
* En interne : `http://localhost:19081/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`

Pour atteindre les ressources exposées par le service, il vous suffit de placer le chemin d’accès à la ressource après le nom du service dans l’URL :

* En externe : `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService/index.html?PartitionKey=3&PartitionKind=Int64Range`
* En interne : `http://localhost:19081/MyApp/MyService/api/users/6?PartitionKey=3&PartitionKind=Int64Range`

La passerelle transmet ensuite ces requêtes à l’URL du service :

* `http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/index.html`
* `http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/api/users/6`

## <a name="special-handling-for-port-sharing-services"></a>Traitement spécial des services de partage de port
Le proxy inverse Service Fabric tente à nouveau de résoudre une adresse de service, et réexécute la requête lorsqu’un service est inaccessible. En règle générale, lorsqu’un service est inaccessible, le réplica ou l’instance de service ont été déplacés vers un autre nœud dans le cadre de leur cycle de vie normal. Dans ce cas, le proxy inverse peut recevoir une erreur de connexion au réseau, qui indique qu’un point de terminaison n’est plus ouvert sur l’adresse résolue à l’origine.

Toutefois, les instances de service ou réplicas peuvent partager un processus hôte, voire un port, lorsqu’ils sont hébergés par un serveur web basé sur HTTP.sys. Cela inclut :

* [System.Net.HttpListener](/dotnet/api/system.net.httplistener?view=netcore-3.1)
* [ASP.NET Core WebListener](https://docs.asp.net/latest/fundamentals/servers.html#weblistener)
* [Katana](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.OwinSelfHost/)

Dans cette situation, le serveur web est probablement disponible dans le processus hôte et répond aux requêtes, mais le réplica ou l’instance de service résolue ne sont plus accessibles sur l’hôte. Dans ce cas, la passerelle reçoit une réponse HTTP 404 du serveur web. Par conséquent, une réponse HTTP 404 peut avoir deux significations :

- 1er cas : l’adresse du service est correcte, mais la ressource demandée par l’utilisateur n’existe pas.
- 2e cas : l’adresse du service est incorrecte, et la ressource demandée par l’utilisateur peut exister sur un autre nœud.

Le premier cas est une erreur HTTP 404 normale, considérée comme une erreur de l’utilisateur. Toutefois, dans le deuxième cas, l’utilisateur a demandé une ressource qui existe réellement. Le proxy inverse n’a pas réussi à la localiser parce que le service a été déplacé. Le proxy inverse doit résoudre à nouveau l’adresse et réexécuter la requête.

Le proxy inverse doit donc faire la distinction entre ces deux cas. Le serveur doit lui fournir une indication lui permettant d’y parvenir.

* Par défaut, le proxy inverse part du principe qu’il s’agit du deuxième cas et tente à nouveau de résoudre le problème, puis de renvoyer la requête.
* Pour indiquer au proxy inverse qu’il s’agit du premier cas, le service doit retourner l’en-tête de réponse HTTP suivant :

  `X-ServiceFabric : ResourceNotFound`

Cet en-tête indique une situation HTTP 404 normale, dans laquelle la ressource demandée n’existe pas, et le proxy inverse ne tente pas de résoudre à nouveau l’adresse du service.

## <a name="special-handling-for-services-running-in-containers"></a>Traitement spécial pour les services s’exécutant dans des conteneurs

Pour les services s’exécutant au sein de conteneurs, vous pouvez utiliser la variable d’environnement `Fabric_NodeIPOrFQDN` pour construire l’[URL de proxy inverse](#uri-format-for-addressing-services-by-using-the-reverse-proxy) comme dans le code suivant :

```csharp
    var fqdn = Environment.GetEnvironmentVariable("Fabric_NodeIPOrFQDN");
    var serviceUrl = $"http://{fqdn}:19081/DockerSFApp/UserApiContainer";
```
Pour le cluster local, `Fabric_NodeIPOrFQDN` a la valeur « localhost » par défaut. Démarrez le cluster local avec le paramètre `-UseMachineName` pour vous assurer que les conteneurs peuvent atteindre le proxy inverse en cours d’exécution sur le nœud. Pour plus d’informations, voir [Configurer votre environnement de développement pour déboguer des conteneurs](service-fabric-how-to-debug-windows-containers.md#configure-your-developer-environment-to-debug-containers).

Les services Service Fabric qui s’exécutent dans des conteneurs Docker Compose nécessitent une configuration spéciale http : ou https : de la *section Ports*  de docker-compose.yml. Pour plus d’informations, consultez [Prise en charge du déploiement Docker Compose dans Azure Service Fabric](service-fabric-docker-compose.md).

## <a name="next-steps"></a>Étapes suivantes
* [Installer et configurer un proxy inverse sur un cluster](service-fabric-reverseproxy-setup.md).
* [Configurer le transfert vers un service HTTP sécurisé avec le proxy inverse](service-fabric-reverseproxy-configure-secure-communication.md)
* [Diagnostiquer des événements de proxy inverse](service-fabric-reverse-proxy-diagnostics.md)
* Pour obtenir un exemple de communication HTTP entre services, consultez cet [exemple de projet sur GitHub](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started).
* [Appels de procédure distante avec Reliable Services à distance](service-fabric-reliable-services-communication-remoting.md)
* [API Web qui utilise OWIN dans Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md)
* [Communication WCF à l’aide de Reliable Services](service-fabric-reliable-services-communication-wcf.md)

[0]: ./media/service-fabric-reverseproxy/external-communication.png
[1]: ./media/service-fabric-reverseproxy/internal-communication.png
