---
title: FAQ sur le développement d’Azure Cache pour Redis
description: Découvrez les réponses aux questions courantes qui vous aideront à développer des solutions pour Azure Cache pour Redis
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 08/06/2020
ms.openlocfilehash: bafd8a9752d2587ec52fe586e442e3bfc86d7537
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585766"
---
# <a name="azure-cache-for-redis-development-faqs"></a>FAQ sur le développement d’Azure Cache pour Redis

Cet article fournit des réponses aux questions courantes sur le développement de solutions pour Azure Cache pour Redis.

## <a name="common-questions-and-answers"></a>Questions et réponses courantes
Cette section aborde les FAQ suivantes :

* [Bien démarrer avec le Cache Azure pour Redis](#how-can-i-get-started-with-azure-cache-for-redis)
* [En quoi consistent les options de configuration StackExchange.Redis ?](#what-do-the-stackexchangeredis-configuration-options-do)
* [Quels clients du Cache Azure pour Redis puis-je utiliser ?](#what-azure-cache-for-redis-clients-can-i-use)
* [Existe-t-il un émulateur local pour le Cache Azure pour Redis ?](#is-there-a-local-emulator-for-azure-cache-for-redis)
* [Comment exécuter des commandes Redis ?](#how-can-i-run-redis-commands)
* [Pourquoi Cache Azure pour Redis n’a-t-il pas de référence de bibliothèque de classes MSDN ?](#why-doesnt-azure-cache-for-redis-have-an-msdn-class-library-reference)
* [Puis-je utiliser le Cache Azure pour Redis comme un cache de session PHP ?](#can-i-use-azure-cache-for-redis-as-a-php-session-cache)
* [Quelles sont les bases de données Redis ?](#what-are-redis-databases)

### <a name="how-can-i-get-started-with-azure-cache-for-redis"></a>Bien démarrer avec le Cache Azure pour Redis
Il existe plusieurs façons de démarrer avec le Cache Azure pour Redis.

* Vous pouvez consulter nos didacticiels disponibles pour [.NET](cache-dotnet-how-to-use-azure-redis-cache.md), [ASP.NET](cache-web-app-howto.md), [Java](cache-java-get-started.md), [Node.js](cache-nodejs-get-started.md) et [Python](cache-python-get-started.md).
* Vous pouvez regarder [Guide pratique pour créer des applications haute performance à l’aide de Cache Microsoft Azure pour Redis](https://azure.microsoft.com/documentation/videos/how-to-build-high-performance-apps-using-microsoft-azure-cache/).
* Vous pouvez consulter la documentation client pour les clients qui correspondent au langage de développement de votre projet pour voir comment utiliser Redis. Il existe de nombreux clients Redis qui peuvent être utilisés avec le Cache Azure pour Redis. Pour obtenir la liste des clients Redis, voir [https://redis.io/clients](https://redis.io/clients).

Si vous n’avez pas encore de compte Azure, vous pouvez :

* [Ouvrir un compte Azure gratuitement](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=redis_cache_hero). Vous obtenez des crédits que vous pouvez utiliser pour essayer des services Azure payants. Même après que les crédits sont épuisés, vous pouvez conserver le compte et utiliser les services et fonctionnalités Azure gratuits.
* [Activez les avantages d’abonnement Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=redis_cache_hero). Votre abonnement MSDN vous donne droit chaque mois à des crédits dont vous pouvez vous servir pour les services Azure payants.

### <a name="what-do-the-stackexchangeredis-configuration-options-do"></a>En quoi consistent les options de configuration StackExchange.Redis ?
StackExchange.Redis présente de nombreuses options. Cette section présente certains des paramètres les plus courants. Pour plus d’informations sur les options de StackExchange.Redis, consultez la page [Configuration StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration).

| ConfigurationOptions | Description | Recommandation |
| --- | --- | --- |
| AbortOnConnectFail |Lorsque la valeur est True, la connexion n’est pas rétablie après une panne réseau. |La valeur False laisse StackExchange.Redis se reconnecter automatiquement . |
| ConnectRetry |Nombre de tentatives de connexion pendant la connexion initiale. |Reportez-vous aux notes suivantes. |
| ConnectTimeout |Délai d’expiration en millisecondes pour les opérations de connexion. |Reportez-vous aux notes suivantes. |

Généralement, les valeurs par défaut du client sont suffisantes. Vous pouvez affiner les options en fonction de votre charge de travail.

* **Nouvelle tentatives**
  * Pour ConnectRetry et ConnectTimeout, la recommandation générale consiste à effectuer un Fail-fast, puis à réessayer. Cela dépend de votre charge de travail et de combien de temps en moyenne votre client prend pour émettre une commande Redis et recevoir une réponse.
  * Laissez StackExchange.Redis se reconnecter automatiquement au lieu de vérifier l’état de la connexion et de vous reconnecter vous-même. **Évitez d’utiliser la propriété ConnectionMultiplexer.IsConnected**.
  * Effet boule de neige : vous pouvez parfois rencontrer un problème, vous réessayez, mais le problème ne fait que grossir et n’est jamais résolu. Dans ce cas, envisagez d’utiliser un algorithme de nouvelle tentative d’interruption exponentiel, comme décrit dans l’article [Conseils généraux sur les nouvelles tentatives](/azure/architecture/best-practices/transient-faults), publié par le groupe Microsoft Patterns & Practices.
  
* **Valeurs de délai d’expiration**
  * Réfléchissez à votre charge de travail et définissez les valeurs en conséquence. Si vous stockez des valeurs élevées, définissez le délai sur une valeur plus élevée.
  * Définissez `AbortOnConnectFail` sur False et laissez StackExchange.Redis se reconnecter pour vous.
  * Utilisez une seule instance de ConnectionMultiplexer pour l’application. Vous pouvez utiliser une LazyConnection pour créer une instance unique qui est renvoyée par une propriété Connection, comme indiqué dans [Connexion au cache avec la classe ConnectionMultiplexer](cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-the-cache).
  * Définissez la propriété `ConnectionMultiplexer.ClientName` sur un nom d’instance unique pour faciliter le diagnostic.
  * Utilisez plusieurs instances `ConnectionMultiplexer` pour les charges de travail personnalisées.
    * Vous pouvez suivre ce modèle si vous savez que la charge de votre application est variable. Par exemple :
    * Vous pouvez avoir un multiplexeur pour gérer les clés de grande taille.
    * Vous pouvez avoir un multiplexeur pour gérer les clés de petite taille.
    * Vous pouvez définir différentes valeurs pour l’expiration du délai et la logique des nouvelles tentatives pour chaque ConnectionMultiplexer que vous utilisez.
    * Définissez la propriété `ClientName` de chaque multiplexeur pour faciliter le diagnostic.
    * Cette recommandation permet d’obtenir une latence optimisée par `ConnectionMultiplexer`.

### <a name="what-azure-cache-for-redis-clients-can-i-use"></a>Quels clients du Cache Azure pour Redis puis-je utiliser ?
L’un des grands avantages de Redis est qu’il existe de nombreux clients prenant en charge de nombreux langages de développement différents. Pour obtenir la liste actuelle des clients, consultez la page des [clients Redis](https://redis.io/clients). Pour obtenir des tutoriels qui couvrent les différents langages et clients, consultez [Utiliser le cache Azure pour Redis](cache-dotnet-how-to-use-azure-redis-cache.md) et les articles connexes dans la table des matières.

[!INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

### <a name="is-there-a-local-emulator-for-azure-cache-for-redis"></a>Existe-t-il un émulateur local pour le Cache Azure pour Redis ?
Il n’existe aucun émulateur local pour le Cache Azure pour Redis, mais vous pouvez exécuter la version MSOpenTech de redis-server.exe à partir des [outils de ligne de commande Redis](https://github.com/MSOpenTech/redis/releases/) sur votre ordinateur local et vous y connecter pour obtenir un résultat similaire à celui d’un émulateur de cache local, comme illustré dans l’exemple suivant :

```csharp
private static Lazy<ConnectionMultiplexer>
    lazyConnection = new Lazy<ConnectionMultiplexer> (() =>
    {
        // Connect to a locally running instance of Redis to simulate
        // a local cache emulator experience.
        return ConnectionMultiplexer.Connect("127.0.0.1:6379");
    });

public static ConnectionMultiplexer Connection
{
    get
    {
        return lazyConnection.Value;
    }
}
```

Vous pouvez éventuellement configurer un fichier [redis.conf](https://redis.io/topics/config) correspondant mieux aux [paramètres de cache par défaut](cache-configure.md#default-redis-server-configuration) de votre Cache Azure pour Redis en ligne le cas échéant.

### <a name="how-can-i-run-redis-commands"></a>Comment exécuter des commandes Redis ?
Vous pouvez utiliser les commandes répertoriées dans [Commandes Redis](https://redis.io/commands#), à l’exception de celles répertoriées dans [Commandes Redis non prises en charge dans le Cache Azure pour Redis](cache-configure.md#redis-commands-not-supported-in-azure-cache-for-redis). Vous disposez de plusieurs options pour exécuter des commandes Redis.

* Si vous avez un cache Standard ou Premium, vous pouvez exécuter les commandes Redis à l’aide de la [console Redis](cache-configure.md#redis-console). Elle offre un moyen sécurisé d'exécuter des commandes Redis dans le portail Azure.
* Vous pouvez aussi utiliser les outils de ligne de commande Redis. Pour ce faire, effectuez les étapes suivantes :
* Téléchargez les [outils de ligne de commande Redis](https://github.com/MSOpenTech/redis/releases/).
* Connectez-vous au cache avec `redis-cli.exe`. Transmettez le point de terminaison du cache à l’aide du commutateur -h et la clé à l’aide de -a, comme indiqué dans l’exemple suivant :
* `redis-cli -h <Azure Cache for Redis name>.redis.cache.windows.net -a <key>`

> [!NOTE]
> Les outils en ligne de commande Redis ne fonctionnent pas avec le port TLS, mais vous pouvez utiliser un utilitaire comme `stunnel` pour connecter de manière sécurisée les outils au port TLS en suivant les instructions données dans l’article [Utiliser l’outil en ligne de commande Redis avec Azure Cache pour Redis](./cache-how-to-redis-cli-tool.md).
>
>

### <a name="why-doesnt-azure-cache-for-redis-have-an-msdn-class-library-reference"></a>Pourquoi Cache Azure pour Redis n’a-t-il pas de référence de bibliothèque de classes MSDN ?
Microsoft Azure Cache pour Redis est dérivé de Redis, un très populaire magasin de données en mémoire open source. Il est accessible par un large éventail de [clients Redis](https://redis.io/clients) pour de nombreux langages de programmation. Chaque client possède sa propre API qui effectue des appels à l’instance de Cache Azure pour Redis à l’aide de [commandes Redis](https://redis.io/commands).

Étant donné que chaque client est différent, il n’y a pas de référence centralisée au sujet des classes sur MSDN. Chaque client a sa propre documentation de référence. En plus de la documentation de référence, il existe plusieurs didacticiels qui montrent comment débuter avec le Cache Azure pour Redis avec plusieurs langues et clients. Pour accéder à ces tutoriels, consultez [Utiliser le cache Azure pour Redis](cache-dotnet-how-to-use-azure-redis-cache.md) et les articles connexes dans la table des matières.

### <a name="can-i-use-azure-cache-for-redis-as-a-php-session-cache"></a>Puis-je utiliser le Cache Azure pour Redis comme un cache de session PHP ?
Oui, pour utiliser le Cache Azure pour Redis comme un cache de session PHP, pointez la chaîne de connexion vers votre instance de Cache Azure pour Redis dans `session.save_path`.

> [!IMPORTANT]
> Lorsque vous utilisez le Cache Azure pour Redis comme un cache de session PHP, vous devez coder l’URL de la clé de sécurité utilisée pour se connecter au cache, comme illustré dans l’exemple suivant :
>
> `session.save_path = "tcp://mycache.redis.cache.windows.net:6379?auth=<url encoded primary or secondary key here>";`
>
> Si la clé n’est pas codée par URL, vous pouvez recevoir une exception avec un message du type : `Failed to parse session.save_path`
>

Pour plus d’informations sur l’utilisation du Cache Azure pour Redis comme un cache de session PHP avec le client PhpRedis, consultez [Gestionnaire PHP Session](https://github.com/phpredis/phpredis#php-session-handler).

### <a name="what-are-redis-databases"></a>Quelles sont les bases de données Redis ?

Les bases de données Redis sont tout simplement une séparation logique des données au sein de la même instance Redis. La mémoire cache est partagée entre toutes les bases de données et la consommation réelle de mémoire d’une base de données dépend des clés/valeurs stockées dans cette base de données. Par exemple, un cache C6 dispose de 53 Go de mémoire et un cache P5 de 120 Go de mémoire. Vous pouvez choisir de placer l’intégralité des 53 Go/120 Go dans une seule base de données, ou vous pouvez les répartir entre plusieurs bases de données. 

> [!NOTE]
> Lorsque vous utilisez un Cache Azure pour Redis Premium avec le clustering activé, seule la base de données 0 est disponible. Cette limitation est une limitation Redis intrinsèque et n’est pas spécifique au Cache Azure pour Redis. Pour plus d’informations, consultez [Dois-je apporter des modifications à mon application cliente pour utiliser le clustering ?](cache-how-to-premium-clustering.md#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
> 
> 

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur d’autres [FAQ sur Azure Cache pour Redis](cache-faq.md).