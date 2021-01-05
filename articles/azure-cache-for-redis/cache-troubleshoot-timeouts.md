---
title: Résoudre les problèmes de dépassement de délai d’expiration avec Azure Cache pour Redis
description: Découvrez comment résoudre les problèmes courants liés au délai d’expiration dans Azure Cache for Redis, tels que les exceptions concernant le délai d’expiration de StackExchange.Redis et des mises à jours correctives du serveur Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 10/18/2019
ms.openlocfilehash: bf8b20dadd2fcd78657aa6877e796b645332dd94
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88213456"
---
# <a name="troubleshoot-azure-cache-for-redis-timeouts"></a>Résoudre les problèmes de dépassement de délai d’expiration avec Azure Cache pour Redis

Cette section traite de la résolution des problèmes de dépassement de délai d’expiration qui se produisent lors de la connexion à Azure Cache pour Redis.

- [Mise à jour corrective du serveur Redis](#redis-server-patching)
- [Exceptions au délai d’expiration de StackExchange.Redis](#stackexchangeredis-timeout-exceptions)

> [!NOTE]
> Dans ce guide, plusieurs procédures de résolution de problèmes comprennent des instructions pour exécuter des commandes Redis et surveiller diverses mesures de performances. Pour plus d’informations et d’instructions, consultez les articles dans la section [Informations supplémentaires](#additional-information) .
>

## <a name="redis-server-patching"></a>Mise à jour corrective du serveur Redis

Azure Cache pour Redis met régulièrement à jour son logiciel serveur dans le cadre de la fonctionnalité de service managé qu’il fournit. Cette activité de [mise à jour corrective ](cache-failover.md) se déroule en grande partie en arrière-plan. Durant les basculements qui ont lieu lors de l’application de correctifs aux nœuds serveurs Redis, les clients Redis connectés à ces nœuds peuvent connaître des dépassements temporaires de délai d’expiration dus à la permutation des connexions entre ces nœuds. Pour plus d’informations sur les effets secondaires potentiels de la mise à jour corrective sur votre application et sur la façon dont vous pouvez améliorer sa gestion des événements de mise à jour corrective, consultez [Quel est l’impact d’un basculement sur mon application cliente ?](cache-failover.md#how-does-a-failover-affect-my-client-application).

## <a name="stackexchangeredis-timeout-exceptions"></a>Exceptions au délai d’expiration de StackExchange.Redis

Pour les opérations synchrones, StackExchange.Redis utilise un paramètre de configuration appelé `synctimeout`, qui a une valeur par défaut de 5000 ms. Si un appel synchrone ne se termine pas en temps voulu, le client StackExchange.Redis génère une erreur de délai d’expiration similaire à la suivante :

```output
    System.TimeoutException: Timeout performing MGET 2728cc84-58ae-406b-8ec8-3f962419f641, inst: 1,mgr: Inactive, queue: 73, qu=6, qs=67, qc=0, wr=1/1, in=0/0 IOCP: (Busy=6, Free=999, Min=2,Max=1000), WORKER (Busy=7,Free=8184,Min=2,Max=8191)
```

Ce message d’erreur contient des mesures qui peuvent vous aider à identifier la cause du problème et à en trouver la solution. Le tableau suivant contient des détails sur les mesures des messages d’erreur.

| Mesure de message d’erreur | Détails |
| --- | --- |
| inst |Au cours du dernier intervalle de temps : 0 commandes ont été émises |
| mgr |Le gestionnaire de sockets fait un `socket.select`, ce qui signifie qu’il demande au système d’exploitation de désigner un socket qui a quelque chose à faire. Le lecteur ne lit pas activement ce qui se trouve sur le réseau, car il pense qu’il n’y a rien à faire. |
| queue |73 opérations sont en cours. |
| qu |6 des opérations en cours se trouvent dans la file d’attente et n’ont pas encore été envoyées au réseau sortant. |
| qs |67 des opérations en cours ont été envoyées au serveur, mais aucune réponse n’est encore disponible. La réponse peut être `Not yet sent by the server` ou `sent by the server but not yet processed by the client.`. |
| qc |Aucune des opérations en cours n’a encore reçu de réponse, mais celles-ci sont marquées comme terminées, car elles attendent dans la boucle d’exécution. |
| wr |Il y a un enregistreur actif (ce qui signifie que les 6 requêtes en attente ne sont pas ignorées) bytes/activewriters |
| commencer |Aucun lecteur n’est actif et aucun octet n’est disponible en lecture sur la carte réseau octets/lecteurs actifs |

Vous pouvez utiliser les étapes suivantes pour rechercher les causes racines possibles.

1. Il est recommandé de suivre la procédure ci-dessous pour se connecter, en cas d’utilisation du client StackExchange.Redis.

    ```csharp
    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        return ConnectionMultiplexer.Connect("cachename.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");

    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }
    ```

    Pour plus d’informations, consultez [Utilisation du Cache Redis Azure](cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-the-cache).

1. Vérifiez que le serveur et l’application cliente se trouvent dans la même région Azure. Par exemple, vous pouvez être confronté à des expirations si votre cache se trouve dans la région USA Est, alors que le client se trouve dans la région USA Ouest et que la requête n’est pas traitée dans l’intervalle `synctimeout`. Vous pouvez également être confronté à des expirations lorsque vous procédez à un débogage sur votre ordinateur de développement local. 

    Il est vivement recommandé de placer le cache et le client dans la même région Azure. Si votre implémentation nécessite des appels interrégionaux, vous devez régler l’intervalle `synctimeout` sur une valeur supérieure à la valeur par défaut (5000 ms), en incluant une propriété `synctimeout` dans la chaîne de connexion. L’exemple suivant montre un extrait de chaîne de connexion pour StackExchange.Redis qui est fournie au cache Azure pour Redis avec une valeur `synctimeout` de 2 000 ms.

    ```output
    synctimeout=2000,cachename.redis.cache.windows.net,abortConnect=false,ssl=true,password=...
    ```

1. Vérifiez que vous utilisez la dernière version en date du [package NuGet StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/). Nous corrigeons en permanence le code pour le rendre plus robuste aux délais d’expiration. Il est donc primordial d’utiliser la dernière version.
1. Si des requêtes sont impactées par les limitations de bande passante sur le serveur ou le client, leur traitement prend davantage de temps, ce qui entraîne des expirations. Pour savoir si le dépassement de délai d’expiration est dû à la bande passante réseau sur le serveur, consultez [Limitation de la bande passante côté serveur](cache-troubleshoot-server.md#server-side-bandwidth-limitation). Pour savoir si le dépassement de délai d’expiration est dû à la bande passante réseau du client, consultez [Limitation de la bande passante côté client](cache-troubleshoot-client.md#client-side-bandwidth-limitation).
1. Constatez-vous une utilisation intensive du processeur sur le serveur ou sur le client ?

   - Vérifiez si le processeur est utilisé de manière intensive sur le client. Une utilisation élevée du processeur peut empêcher la requête d’être traitée dans l’intervalle `synctimeout` et entraîner son expiration. Augmentez la taille de votre client ou répartissez la charge pour résoudre ce problème.
   - Examinez si le processeur est utilisé de manière intensive sur le serveur à l’aide de la [métrique de niveau de performance du cache](cache-how-to-monitor.md#available-metrics-and-reporting-intervals). Si des requêtes sont reçues pendant que Redis utilise intensivement le processeur, ces requêtes peuvent expirer. Pour résoudre ce problème, vous pouvez répartir la charge sur plusieurs partitions dans un cache premium, ou effectuer une mise à niveau vers une taille ou un niveau de tarification supérieur. Pour plus d’informations, consultez [Limitation de la bande passante côté serveur](cache-troubleshoot-server.md#server-side-bandwidth-limitation).
1. Le traitement de certaines commandes est-il trop long sur le serveur ? Les commandes dont l’exécution prend beaucoup de temps sur le serveur Redis peuvent entraîner des expirations. Pour plus d’informations, consultez [Commandes de longue durée](cache-troubleshoot-server.md#long-running-commands). Vous pouvez vous connecter à votre instance du Cache Azure pour Redis à l’aide du client redis-cli ou de la [console Redis](cache-configure.md#redis-console). Ensuite, exécutez la commande [SLOWLOG](https://redis.io/commands/slowlog) pour voir si l’exécution des requêtes est plus lente que d’habitude. Le serveur Redis et StackExchange.Redis sont optimisés pour traiter un grand nombre de petites requêtes plutôt qu’un nombre réduit de demandes volumineuses. Fractionner vos données en segments plus petits peut améliorer les choses.

    Pour plus d’informations sur la connexion au point de terminaison TLS/SSL du cache à l’aide de redis-cli et de stunnel, consultez le billet de blog [Announcing ASP.NET Session State Provider for Redis Preview Release](https://devblogs.microsoft.com/aspnet/announcing-asp-net-session-state-provider-for-redis-preview-release/).
1. Une charge élevée du serveur Redis peut provoquer des délais d’expiration. Vous pouvez superviser la charge du serveur à l’aide de la [métrique des performances du cache](cache-how-to-monitor.md#available-metrics-and-reporting-intervals) `Redis Server Load`. Une charge serveur de 100 (valeur maximale) signifie que le serveur redis a été occupé à plein temps à traiter des demandes. Pour voir si certaines demandes monopolisent les capacités du serveur, exécutez la commande SlowLog comme indiqué dans le paragraphe précédent. Pour plus d’informations, consultez Utilisation élevée du processeur / charge importante du serveur.
1. Un autre événement côté client a-t-il pu causer un blocage du réseau ? Les événements courants incluent notamment le scale-up et le scale-down du nombre d’instances de client, le déploiement de nouvelles versions du client et la mise à l’échelle automatique. Durant nos tests, nous avons découvert que la mise à l’échelle automatique ainsi que le scale-up et le scale-down pouvaient entraîner une perte de la connectivité réseau sortante pendant plusieurs secondes. Le code de StackExchange.Redis résiste à de tels événements et rétablit la connexion. Pendant la reconnexion, les requêtes qui se trouvent dans la file d’attente peuvent expirer.
1. Dans le cache, une requête volumineuse précédant plusieurs petites requêtes a-t-elle expiré ? Le paramètre `qs` du message d’erreur vous indique le nombre de requêtes envoyées du client au serveur, mais qui n’ont pas encore traité de réponse. Cette valeur peut continuer d’augmenter car StackExchange.Redis utilise une seule connexion TCP et ne peut lire qu’une réponse à la fois. Même si la première opération a expiré, cela n’empêche pas l’envoi d’autres données vers ou à partir du serveur. Les autres requêtes sont donc bloquées jusqu’à ce que la requête volumineuse ait été traitée, ce qui peut entraîner des expirations. Une solution pour réduire le risque de délais d’expiration consiste à vérifier que le cache est assez grand pour votre charge de travail et à fractionner les valeurs importantes en valeurs plus petites. Une autre solution consiste à utiliser un pool d’objets `ConnectionMultiplexer` dans votre client et à choisir le `ConnectionMultiplexer` le moins chargé lors de l’envoi d’une nouvelle demande. Un chargement sur plusieurs objets de connexion permet d’empêcher une expiration de faire expirer les autres requêtes.
1. Si vous utilisez `RedisSessionStateProvider`, vérifiez que vous avez configuré correctement le délai d’expiration de nouvelle tentative. `retryTimeoutInMilliseconds` doit être supérieur à `operationTimeoutInMilliseconds`, sans quoi aucune nouvelle tentative ne se produit. Dans l’exemple suivant, `retryTimeoutInMilliseconds` est réglé sur 3 000. Pour plus d’informations, consultez la page [Fournisseur d’état de session ASP.NET pour cache Azure pour Redis](cache-aspnet-session-state-provider.md) et [How to use the configuration parameters of Session State Provider and Output Cache Provider](https://github.com/Azure/aspnet-redis-providers/wiki/Configuration) (Comment utiliser les paramètres de configuration du fournisseur d’état de session et du fournisseur de caches de sortie).

    ```xml
    <add
      name="AFRedisCacheSessionStateProvider"
      type="Microsoft.Web.Redis.RedisSessionStateProvider"
      host="enbwcache.redis.cache.windows.net"
      port="6380"
      accessKey="…"
      ssl="true"
      databaseId="0"
      applicationName="AFRedisCacheSessionState"
      connectionTimeoutInMilliseconds = "5000"
      operationTimeoutInMilliseconds = "1000"
      retryTimeoutInMilliseconds="3000" />
    ```

1. Vérifiez l’utilisation de la mémoire sur le serveur Azure Cache for Redis en [supervisant](cache-how-to-monitor.md#available-metrics-and-reporting-intervals) `Used Memory RSS` et `Used Memory`. Si une stratégie d’éviction est en place, Redis commence la suppression de clés lorsque `Used_Memory` atteint la taille du cache. Dans l’idéal, `Used Memory RSS` doit être légèrement supérieur à `Used memory`. Une différence importante traduit une fragmentation de la mémoire (interne ou externe). Lorsque `Used Memory RSS` est inférieur à `Used Memory`, cela signifie que le système d’exploitation a permuté une partie de la mémoire cache. Si cette permutation se produit, vous pouvez rencontrer des latences importantes. Comme Redis ne contrôle pas le mode de mappage de ses allocations aux pages mémoire, une valeur élevée de `Used Memory RSS` traduit souvent un pic d’utilisation de la mémoire. Lorsque le serveur Redis libère de la mémoire, l’allocateur prend de la mémoire, mais il peut ne pas la rendre au système. Il peut y avoir une différence entre la valeur de `Used Memory` et la consommation de mémoire telle qu’elle est indiquée par le système d’exploitation. La mémoire a peut-être été utilisée et publiée par Redis, mais elle n’a pas été rendue au système. Pour réduire les problèmes de mémoire, vous pouvez effectuer les étapes suivantes :

   - Augmentez la taille du cache, afin de ne plus être soumis aux limitations de mémoire du système.
   - Définissez des délais d’expiration sur les clés afin que les anciennes valeurs soient supprimées de manière proactive.
   - Surveillez la mesure de cache `used_memory_rss`. Lorsque cette valeur s’approche de la taille du cache, vous risquez de voir apparaître des problèmes de performance. Répartissez les données sur plusieurs partitions si vous utilisez un cache Premium, ou augmentez la taille du cache.

   Pour plus d’informations, consultez [Saturation de la mémoire sur le serveur Redis](cache-troubleshoot-server.md#memory-pressure-on-redis-server).

## <a name="additional-information"></a>Informations supplémentaires

- [Résoudre les problèmes côté client liés à Azure Cache pour Redis](cache-troubleshoot-client.md)
- [Résoudre les problèmes côté serveur liés à Azure Cache pour Redis](cache-troubleshoot-server.md)
- [Comment puis-je évaluer et tester les performances de mon cache ?](cache-management-faq.md#how-can-i-benchmark-and-test-the-performance-of-my-cache)
- [Surveillance du cache Azure pour Redis](cache-how-to-monitor.md)
