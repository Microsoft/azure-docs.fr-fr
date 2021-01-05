---
title: FAQ sur la gestion d’Azure Cache pour Redis
description: Découvrez les réponses aux questions fréquentes qui vous aideront à gérer Azure Cache pour Redis
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 08/06/2020
ms.openlocfilehash: 15c7ed4ca9d04e4bb314eea8b92bef749d2369b1
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92537658"
---
# <a name="azure-cache-for-redis-management-faqs"></a>FAQ sur la gestion d’Azure Cache pour Redis
Cet article fournit des réponses aux questions fréquentes sur la gestion d’Azure Cache pour Redis.

## <a name="common-questions-and-answers"></a>Questions et réponses courantes
Cette section aborde les FAQ suivantes :

* [Quand dois-je activer le port non TLS/SSL pour la connexion à Redis ?](#when-should-i-enable-the-non-tlsssl-port-for-connecting-to-redis)
* [Quelles sont les meilleures pratiques en matière de production ?](#what-are-some-production-best-practices)
* [Quels sont les points à prendre en compte en ce qui concerne l’utilisation des commandes Redis courantes ?](#what-are-some-of-the-considerations-when-using-common-redis-commands)
* [Comment puis-je évaluer et tester les performances de mon cache ?](#how-can-i-benchmark-and-test-the-performance-of-my-cache)
* [Informations importantes sur la croissance du pool de threads](#important-details-about-threadpool-growth)
* [Activer le nettoyage de la mémoire (GC) du serveur pour obtenir un meilleur débit sur le client lors de l’utilisation de StackExchange.Redis](#enable-server-gc-to-get-more-throughput-on-the-client-when-using-stackexchangeredis)
* [Considérations relatives aux performances de connexion](#performance-considerations-around-connections)

### <a name="when-should-i-enable-the-non-tlsssl-port-for-connecting-to-redis"></a>Quand dois-je activer le port non TLS/SSL pour la connexion à Redis ?
Le serveur Redis ne prend pas en charge TLS de façon native, contrairement à Azure Cache pour Redis. Si vous vous connectez à Azure Cache pour Redis et que votre client est compatible avec TLS, par exemple StackExchange.Redis, vous devez utiliser TLS.

>[!NOTE]
>Le port non TLS est désactivé par défaut pour les nouvelles instances Azure Cache pour Redis. Si votre client ne prend pas en charge TLS, vous devez activer le port non TLS en suivant les instructions de la section [Ports d’accès](cache-configure.md#access-ports) de l’article [Configuration d’un cache dans Azure Cache pour Redis](cache-configure.md).
>
>

Des outils Redis tels que `redis-cli` ne fonctionnent pas avec le port TLS, mais vous pouvez utiliser un utilitaire comme `stunnel` pour connecter en toute sécurité les outils au port TLS en suivant les instructions du billet de blog [Annonce du fournisseur d’état de session ASP.NET pour la version préliminaire de Redis](https://devblogs.microsoft.com/aspnet/announcing-asp-net-session-state-provider-for-redis-preview-release/).

Pour obtenir des instructions sur le téléchargement des outils Redis, consultez la section [Comment exécuter des commandes Redis ?](cache-development-faq.md#how-can-i-run-redis-commands) .

### <a name="what-are-some-production-best-practices"></a>Quelles sont les meilleures pratiques en matière de production ?
* [Meilleures pratiques StackExchange.Redis](#stackexchangeredis-best-practices)
* [Configuration et concepts](#configuration-and-concepts)
* [Tests de performances](#performance-testing)

#### <a name="stackexchangeredis-best-practices"></a>Meilleures pratiques StackExchange.Redis
* Définissez `AbortConnect` sur false, puis laissez le ConnectionMultiplexer se reconnecter automatiquement. [Vous trouverez plus d’informations ici](https://gist.github.com/JonCole/36ba6f60c274e89014dd#file-se-redis-setabortconnecttofalse-md).
* Réutilisez le ConnectionMultiplexer, ne créez pas de nouvelle instance pour chaque requête. Le modèle `Lazy<ConnectionMultiplexer>`[illustré ici](cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-the-cache) est recommandé.
* Redis fonctionne de manière optimale avec des valeurs plus petites et il est donc recommandé de fractionner les données plus volumineuses dans plusieurs clés. Dans [cette discussion Redis](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ), la valeur 100 Ko est considérée comme volumineuse. Listez [cet article](https://gist.github.com/JonCole/db0e90bedeb3fc4823c2#large-requestresponse-size) illustrant un exemple de problème pouvant être provoqué par des valeurs élevées.
* Configurez vos [paramètres ThreadPool](#important-details-about-threadpool-growth) pour éviter les délais d’attente.
* Utilisez au moins la valeur par défaut connectTimeout de 5 secondes. Cet intervalle laisse à StackExchange.Redis suffisamment de temps pour rétablir la connexion en cas de blocage du réseau.
* Gardez à l’esprit les coûts de performances associés aux différentes opérations que vous exécutez. Par exemple, la commande `KEYS` est une opération O(n) et doit être évitée. Le [site redis.io](https://redis.io/commands/) fournit plus d’informations sur la complexité de temps de chaque opération prise en charge. Cliquez sur chaque commande pour afficher la complexité de chaque opération.

#### <a name="configuration-and-concepts"></a>Configuration et concepts
* Utilisez le niveau Standard ou Premium pour les systèmes de production. Le niveau De base est un système à nœud unique, sans réplication des données ni contrat SLA. En outre, utilisez au moins un cache C1. Les caches C0 s’appliquent généralement aux scénarios de développement/test simples.
* N’oubliez pas que Redis est un magasin de données **In-Memory** . Lisez [cet article](https://gist.github.com/JonCole/b6354d92a2d51c141490f10142884ea4#file-whathappenedtomydatainredis-md) pour identifier les scénarios où une perte de données peut se produire.
* Développez votre système de telle sorte qu’il puisse gérer les problèmes de connexion [liés à une mise à jour corrective et à un basculement](https://gist.github.com/JonCole/317fe03805d5802e31cfa37e646e419d#file-azureredis-patchingexplained-md).

#### <a name="performance-testing"></a>Tests de performances
* Commencez par utiliser `redis-benchmark.exe` pour avoir une idée du débit possible avant d’effectuer vos propres tests de performances. `redis-benchmark` ne prenant pas en charge TLS, vous devez [activer le port non TLS via le portail Azure](cache-configure.md#access-ports) avant d’exécuter le test. Par exemple, consultez la rubrique [Comment puis-je évaluer et tester les performances de mon cache ?](#how-can-i-benchmark-and-test-the-performance-of-my-cache)
* La machine virtuelle cliente utilisée pour le test doit figurer dans la même région que votre instance de Cache Azure pour Redis.
* Nous recommandons d’utiliser des machines virtuelles Dv2 pour votre client car elles disposent d’un matériel plus performant et fournissent de meilleurs résultats.
* Assurez-vous que la machine virtuelle cliente que vous choisissez possède au moins autant de puissance de calcul et de bande passante que le cache que vous testez.
* Activez VRSS sur la machine cliente si vous êtes sous Windows. [Vous trouverez plus d’informations ici](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn383582(v=ws.11)).
* Les instances Redis de niveau Premium offriront une meilleure latence de réseau et un débit supérieur car elles s’exécutent sur un matériel plus performant en termes de processeur et de réseau.

### <a name="what-are-some-of-the-considerations-when-using-common-redis-commands"></a>Quels sont les points à prendre en compte en ce qui concerne les commandes Redis les plus courantes ?

* Évitez d’utiliser certaines commandes Redis qui durent longtemps, sauf si vous comprenez pleinement leur impact. Par exemple, n’exécutez pas la commande [KEYS](https://redis.io/commands/keys) en production. Selon le nombre de clés, la commande peut prendre beaucoup de temps pour s’exécuter. Redis est un serveur à thread unique qui traite une commande à la fois. Si vous avez émis d’autres commandes après KEYS, elles ne sont pas traitées tant que Redis traite la commande KEYS. Le [site redis.io](https://redis.io/commands/) fournit plus d’informations sur la complexité de temps de chaque opération prise en charge. Cliquez sur chaque commande pour afficher la complexité de chaque opération.
* Taille des clés : dois-je utiliser des petites clés/valeurs ou de grandes clés/valeurs ? Cela dépend du scénario. Si votre scénario nécessite des clés plus longues, vous pouvez ajuster les valeurs ConnectionTimeout, puis les réessayer avant d’adapter votre logique de nouvelle tentative. Du point de vue du serveur Redis, les valeurs les plus petites donnent de meilleures performances.
* Cela ne veut pas dire que vous ne pouvez pas stocker de plus grandes valeurs dans Redis. Vous devez simplement être au fait de ces informations. Les temps de latence seront plus élevés. Si vous avez un jeu de données plus grand et un autre plus petit, vous pouvez utiliser plusieurs instances ConnectionMultiplexer, chacune configurée avec un ensemble différent de valeurs de délai d’expiration et de nouvelles tentatives, comme décrit dans la section précédente [En quoi consistent les options de configuration StackExchange.Redis](cache-development-faq.md#what-do-the-stackexchangeredis-configuration-options-do).

### <a name="how-can-i-benchmark-and-test-the-performance-of-my-cache"></a>Comment puis-je évaluer et tester les performances de mon cache ?
* [Activez les diagnostics du cache](cache-how-to-monitor.md#enable-cache-diagnostics) afin de pouvoir [surveiller](cache-how-to-monitor.md) l’intégrité de votre cache. Vous pouvez afficher les mesures dans le portail Azure, et [les télécharger et les analyser](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring) à l’aide des outils de votre choix.
* Vous pouvez utiliser redis-benchmark.exe pour tester la charge de votre serveur Redis.
* Assurez-vous que le client de test de la charge et le Cache Azure pour Redis se trouvent dans la même région.
* Utilisez redis-cli.exe et surveillez le cache à l’aide de la commande INFO.
* Si votre charge provoque une fragmentation élevée de la mémoire, vous devez augmenter la taille de votre cache.
* Pour obtenir des instructions sur le téléchargement des outils Redis, consultez la section [Comment exécuter des commandes Redis ?](cache-development-faq.md#how-can-i-run-redis-commands) .

Les commandes suivantes fournissent un exemple d'utilisation de redis-benchmark.exe. Pour obtenir des résultats précis, exécutez ces commandes à partir d’une machine virtuelle située dans la même région que votre cache.

* Test de requêtes SET en pipeline à l’aide d’une charge utile de 1 ko

  `redis-benchmark.exe -h **yourcache**.redis.cache.windows.net -a **yourAccesskey** -t SET -n 1000000 -d 1024 -P 50`
* Testez des requêtes GET en pipeline à l’aide d’une charge utile de 1 ko.

>[!NOTE]
> Exécutez le jeu SET indiqué ci-dessus pour remplir le cache
>

  `redis-benchmark.exe -h **yourcache**.redis.cache.windows.net -a **yourAccesskey** -t GET -n 1000000 -d 1024 -P 50`

### <a name="important-details-about-threadpool-growth"></a>Informations importantes sur la croissance du pool de threads
Le pool de threads CLR comporte deux types de threads : « Worker » et « I/O Completion Port » (IOCP).

* Les threads de travail sont utilisés notamment pour le traitement des méthodes `Task.Run(…)` ou `ThreadPool.QueueUserWorkItem(…)`. Ces threads sont également utilisés par différents composants dans le CLR lorsqu’une tâche doit être effectuée sur un thread en arrière-plan.
* Les threads IOCP sont utilisés dans le cas d’E/S asynchrones, comme la lecture à partir du réseau.

Le pool de threads fournit de nouveaux threads Worker ou IOCP à la demande (sans aucune limitation) jusqu’à ce qu’il atteigne le paramètre « Minimum » pour chaque type de thread. Par défaut, le nombre minimal de threads est défini sur le nombre de processeurs d’un système.

Une fois que le nombre de threads existants (occupés) atteint le nombre « minimal » de threads, le pool de threads limite le taux d’injection des nouveaux threads à hauteur d’un thread toutes les 500 millisecondes. Généralement, si votre système reçoit une rafale de tâches nécessitant un thread IOCP, la tâche sera traitée rapidement. Toutefois, si le nombre de tâches est supérieur au paramètre « Minimum » configuré, le traitement de certaines tâches sera retardé car le ThreadPool attendra l’une ou l’autre des conditions suivantes :

* Un thread existant se libère pour traiter la tâche.
* Aucun thread existant ne se libère pendant 500 ms, auquel cas un nouveau thread est créé.

En fait, cela signifie que lorsque le nombre de threads occupés est supérieur au nombre minimum de threads, vous devrez probablement subir un délai de 500 ms avant que le trafic réseau ne soit traité par l’application. Il est également important de noter que lorsqu’un thread existant reste inactif pendant plus de 15 secondes (d’après mes souvenirs), il sera nettoyé et ce cycle de croissance et de diminution peut se répéter.

Observons cet exemple de message d’erreur de StackExchange.Redis (build 1.0.450 ou version ultérieure) ; vous voyez que les statistiques de pool de threads s’affichent désormais (voir les détails IOCP et WORKER ci-dessous).

```
System.TimeoutException: Timeout performing GET MyKey, inst: 2, mgr: Inactive,
queue: 6, qu: 0, qs: 6, qc: 0, wr: 0, wq: 0, in: 0, ar: 0,
IOCP: (Busy=6,Free=994,Min=4,Max=1000),
WORKER: (Busy=3,Free=997,Min=4,Max=1000)
```

Dans l’exemple précédent, vous pouvez voir que six threads sont occupés pour le thread IOCP et que le système est configuré pour autoriser au minimum quatre threads. Dans ce cas, le client subirait probablement deux retards de 500 ms car 6 > 4.

Notez que StackExchange.Redis peut atteindre les délais d’expiration si la croissance des threads IOCP ou WORKER est limitée.

#### <a name="recommendation"></a>Recommandation

Par conséquent, nous recommandons fortement aux clients de définir la valeur de configuration minimale pour les threads IOCP et WORKER sur une valeur supérieure à la valeur par défaut. Il est impossible de recommander une valeur unique car une valeur parfaitement adaptée à une application sera probablement trop élevée ou trop faible pour une autre application. Ce paramètre peut également avoir un impact sur les performances d’autres composants d’applications complexes, ce qui signifie que chaque client doit ajuster ce paramètre en fonction de ses besoins spécifiques. Le mieux est de commencer à 200 ou 300, puis de tester et d’ajuster cette valeur en fonction des besoins.

Configuration de ce paramètre :

* Nous vous recommandons de modifier ce paramètre programmatiquement à l’aide de la méthode [ThreadPool.SetMinThreads (...)](/dotnet/api/system.threading.threadpool.setminthreads#System_Threading_ThreadPool_SetMinThreads_System_Int32_System_Int32_) dans `global.asax.cs`. Par exemple :

    ```csharp
    private readonly int minThreads = 200;
    void Application_Start(object sender, EventArgs e)
    {
        // Code that runs on application startup
        AreaRegistration.RegisterAllAreas();
        RouteConfig.RegisterRoutes(RouteTable.Routes);
        BundleConfig.RegisterBundles(BundleTable.Bundles);
        ThreadPool.SetMinThreads(minThreads, minThreads);
    }
    ```

    > [!NOTE]
    > La valeur spécifiée par cette méthode est un paramètre global qui affecte tout le domaine d’application. Par exemple, si vous avez une machine à 4 cœurs et que vous souhaitez définir *minWorkerThreads* et *minIoThreads* à 50 par processeur pendant l’exécution, vous utiliserez **ThreadPool.SetMinThreads(200, 200)** .

* Il est également possible de spécifier le paramètre minimal de threads à l’aide du [paramètre de configuration *minIoThreads* ou *minWorkerThreads*](/previous-versions/dotnet/netframework-4.0/7w2sway1(v=vs.100)) sous l’élément de configuration `<processModel>` dans `Machine.config`, généralement situé dans `%SystemRoot%\Microsoft.NET\Framework\[versionNumber]\CONFIG\`. **Définir le nombre minimal de threads de cette manière n’est généralement pas recommandé, car il s’agit d’un paramètre de l’échelle du système.**

  > [!NOTE]
  > La valeur spécifiée dans cet élément de configuration est un paramètre applicable *par cœur* . Par exemple, si vous utilisez un ordinateur à 4 cœurs et que vous souhaitez définir votre paramètre *minIoThreads* sur 200 au moment de l’exécution, vous utilisez `<processModel minIoThreads="50"/>`.
  >

### <a name="enable-server-gc-to-get-more-throughput-on-the-client-when-using-stackexchangeredis"></a>Activation de Garbage Collection sur le serveur pour obtenir un meilleur début sur le client lors de l'utilisation de StackExchange.Redis
L'activation de Garbage Collection sur le serveur peut optimiser le client, fournir des performances et un débit optimaux lors de l'utilisation de StackExchange.Redis. Pour plus d'informations sur Garbage Collection sur le serveur et son activation, consultez les articles suivants :

* [Activation de Garbage Collection sur le serveur](/dotnet/framework/configure-apps/file-schema/runtime/gcserver-element)
* [Principes de base du Garbage Collection](/dotnet/standard/garbage-collection/fundamentals)
* [Garbage Collection et performances](/dotnet/standard/garbage-collection/performance)

### <a name="performance-considerations-around-connections"></a>Considérations relatives aux performances de connexion

Chaque niveau tarifaire est associé à des limites spécifiques concernant les connexions clientes, la mémoire et la bande passante. Si chaque taille de cache autorise *jusqu’à* un certain nombre de connexions, chaque connexion à Redis entraîne une surcharge. Un exemple d’une telle surcharge est l’utilisation du processeur et de la mémoire à la suite d’un chiffrement TLS/SSL. La limite maximale de connexions pour une taille de cache donnée suppose un cache peu chargé. Si la charge de la surcharge de connexion *plus* la charge des opérations de client dépasse la capacité du système, le cache peut rencontrer des problèmes de capacité, même si vous n’avez pas dépassé la limite de connexion pour la taille de cache actuelle.

Pour plus d’informations sur les différentes limites de connexion imposées sur chaque niveau, consultez la page [Tarification du Cache Azure pour Redis](https://azure.microsoft.com/pricing/details/cache/). Pour plus d’informations sur les connexions et les autres configurations par défaut, consultez la page [Configuration du serveur Redis par défaut](cache-configure.md#default-redis-server-configuration).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur d’autres [FAQ sur Azure Cache pour Redis](cache-faq.md).