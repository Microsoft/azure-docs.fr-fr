---
title: Configurer le clustering Redis - Azure Cache for Redis Premium
description: Découvrez comment créer et gérer le clustering Redis pour vos instances de niveau Premium de Cache Azure pour Redis
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 10/09/2020
ms.openlocfilehash: 9545dd1480b9d16285d936787cf37fc087e882e1
ms.sourcegitcommit: 090ea6e8811663941827d1104b4593e29774fa19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "92000046"
---
# <a name="how-to-configure-redis-clustering-for-a-premium-azure-cache-for-redis"></a>Comment configurer le clustering pour le niveau Premium de Cache Azure pour Redis
Le Cache Azure pour Redis offre différents types de caches permettant de choisir parmi plusieurs tailles et fonctionnalités de caches, notamment les fonctionnalités de niveau Premium telles que le clustering, la persistance et la prise en charge du réseau virtuel. Cet article explique comment configurer le clustering dans une instance de niveau Premium de Cache Azure pour Redis.

## <a name="what-is-redis-cluster"></a>Qu’est-ce que le cluster Redis ?
Cache Azure pour Redis propose le cluster Redis tel qu’[implémenté dans Redis](https://redis.io/topics/cluster-tutorial). Avec le cluster Redis, vous bénéficiez des avantages suivants : 

* Possibilité de fractionner automatiquement votre dataset parmi plusieurs nœuds. 
* Continuité des opérations quand un sous-ensemble de nœuds rencontre des erreurs ou ne peut pas communiquer avec le reste du cluster. 
* Débit supplémentaire : le débit augmente de façon linéaire à mesure que vous augmentez le nombre de partitions. 
* Taille de mémoire supplémentaire : augmente de façon linéaire à mesure que vous augmentez le nombre de partitions.  

Le clustering n’augmente pas le nombre de connexions disponibles pour un cache en cluster. Pour plus d’informations sur la taille, le débit et la bande passante des caches Premium, consultez [Choisir le bon niveau de service](cache-overview.md#choosing-the-right-tier)

Dans Azure, le cluster Redis est proposé sous forme de modèle principal/réplica où chaque partition a une paire principal/réplica avec réplication gérée par le service Cache Azure pour Redis. 

## <a name="clustering"></a>Clustering
La configuration du clustering s’effectue dans le panneau **Nouveau cache Azure pour Redis** lors de la création du cache. 

1. Pour créer un cache Premium, connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Créer une ressource**. En plus de créer des caches dans le portail Azure, vous pouvez en créer à l’aide de modèles Resource Manager, PowerShell ou Azure CLI. Pour plus d’informations sur la création d’un cache Azure pour Redis, consultez la section [Création d’un cache](cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).

    :::image type="content" source="media/cache-private-link/1-create-resource.png" alt-text="Créer une ressource.":::
   
2. Dans la page **Nouvelle**, sélectionnez **Bases de données**, puis **Azure Cache pour Redis**.

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Créer une ressource.":::

3. Dans la page **Nouveau cache Redis**, configurez les paramètres du nouveau cache Premium.
   
   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Nom DNS** | Entrez un nom globalement unique. | Le nom du cache doit être une chaîne de 1 à 63 caractères ne contenant que des chiffres, des lettres et des traits d’union. Le nom doit commencer et se terminer par un chiffre ou une lettre, et ne peut pas contenir de traits d’union consécutifs. Le *nom d’hôte* de votre instance de cache sera *\<DNS name>.redis.cache.windows.net*. | 
   | **Abonnement** | Dans la liste déroulante, sélectionnez votre abonnement. | Abonnement sous lequel créer cette nouvelle instance d’Azure Cache pour Redis. | 
   | **Groupe de ressources** | Dans la liste déroulante, sélectionnez un groupe de ressources ou choisissez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre cache et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
   | **Lieu** | Dans la liste déroulante, sélectionnez un emplacement. | Choisissez une [Région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre cache. |
   | **Type de cache** | Dans la liste déroulante, sélectionnez un cache Premium pour configurer les fonctionnalités Premium. Pour plus d’informations, consultez la page [Tarification Azure Cache pour Redis](https://azure.microsoft.com/pricing/details/cache/). |  Le niveau tarifaire détermine la taille, les performances et les fonctionnalités disponibles pour le cache. Pour plus d’informations, consultez [Présentation du cache Azure pour Redis](cache-overview.md). |

4. Sélectionnez l’onglet **Réseau** ou cliquez sur le bouton **Réseau** au bas de la page.

5. Sous l’onglet **Réseau**, sélectionnez votre méthode de connectivité. Pour les instances de cache Premium, vous pouvez vous connecter de manière publique, via des adresses IP ou des points de terminaison de service publics, ou de manière privée, à l’aide d’un point de terminaison privé.

6. Sélectionnez le bouton **Suivant : Avancé** ou cliquez sur le bouton **Suivant : Avancé** en bas de la page.

7. Sous l’onglet **Avancé** d’une instance de cache Premium, configurez les paramètres pour le port non TLS, le clustering et la persistance des données. Pour activer le clustering, cliquez sur **Activer**.

    :::image type="content" source="media/cache-how-to-premium-clustering/redis-cache-clustering.png" alt-text="Créer une ressource.":::

    Vous pouvez avoir jusqu’à 10 partitions dans le cluster. Après avoir cliqué sur **Activer**, faites glisser le curseur ou saisissez un nombre compris entre 1 et 10 pour **Nombre de partitions**, puis cliquez sur **OK**.

    Chaque partition est une paire de cache principal/réplica gérée par Azure et la taille totale du cache est calculée en multipliant le nombre de partitions par la taille de cache sélectionnée dans le niveau tarifaire.

    :::image type="content" source="media/cache-how-to-premium-clustering/redis-cache-clustering-selected.png" alt-text="Créer une ressource.":::

    Une fois le cache créé, vous vous y connectez et l’utilisez tout comme un cache hors cluster. Redis distribue les données parmi les partitions de Cache. Si les diagnostics sont [activés](cache-how-to-monitor.md#enable-cache-diagnostics), les mesures sont capturées séparément pour chaque partition et peuvent être [affichées](cache-how-to-monitor.md) dans le panneau Cache Azure pour Redis. 

8. Sélectionnez le bouton **Suivant : Étiquettes** ou cliquez sur le bouton **Suivant : Étiquettes** au bas de la page.

9. Si vous le voulez, sous l’onglet **Étiquettes**, entrez le nom et la valeur si vous souhaitez catégoriser la ressource. 

10. Sélectionnez **Vérifier + créer**. Vous êtes redirigé vers l’onglet Vérifier + créer où Azure valide votre configuration.

11. Une fois que le message vert Validation réussie s’affiche, sélectionnez **Créer**.

La création du cache prend un certain temps. Vous pouvez surveiller la progression dans la page **Vue d’ensemble** d’Azure Cache pour Redis. Lorsque **État** indique **En cours d’exécution**, le cache est prêt pour utilisation. 

> [!NOTE]
> 
> Il doit exister quelques différences mineures dans votre application cliente lorsque le clustering est configuré. Pour en savoir plus, voir, [Dois-je apporter des modifications à mon application cliente pour utiliser le clustering ?](#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
> 
> 

Pour accéder à un exemple de code relatif à l’utilisation du clustering avec le client StackExchange.Redis, consultez la partie [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) de l’exemple [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).

<a name="cluster-size"></a>

## <a name="change-the-cluster-size-on-a-running-premium-cache"></a>Modifier la taille de cluster sur un cache de niveau Premium en cours d’exécution
Pour modifier la taille de cluster d’un cache de niveau Premium en cours d’exécution sur lequel le clustering est activé, cliquez sur **Taille du cluster** dans le **menu Ressource**.

![Taille du cluster Redis][redis-cache-redis-cluster-size]

Pour modifier la taille du cluster, utilisez le curseur ou entrez un nombre compris entre 1 et 10 dans la zone de texte **Nombre de partitions**, puis cliquez sur **OK** pour enregistrer.

L’augmentation de la taille du cluster a pour effet d’augmenter le débit maximal et la taille du cache. L’augmentation de la taille du cluster n’augmente pas le nombre maximal de connexions disponibles pour les clients.

> [!NOTE]
> La mise à l’échelle d’un cluster exécute la commande [MIGRATE](https://redis.io/commands/migrate), qui est une commande coûteuse. Par conséquent, pour minimiser l’impact, envisagez d’exécuter cette opération pendant les heures creuses. Pendant le processus de migration, vous observez un pic de charge du serveur. La mise à l’échelle d’un cluster est un processus long et le temps nécessaire dépend du nombre de clés et de la taille des valeurs associées à celles-ci.
> 
> 

## <a name="clustering-faq"></a>Forum aux questions sur le clustering
La liste suivante présente les réponses aux questions les plus fréquemment posées sur la persistance du clustering de Cache Azure pour Redis.

* [Dois-je apporter des modifications à mon application cliente pour utiliser le clustering ?](#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
* [Comment les clés sont-elles distribuées dans un cluster ?](#how-are-keys-distributed-in-a-cluster)
* [Quelle est la taille de cache la plus grande que je peux créer ?](#what-is-the-largest-cache-size-i-can-create)
* [Tous les clients Redis prennent-ils en charge le clustering ?](#do-all-redis-clients-support-clustering)
* [Comment puis-je me connecter à mon cache quand le clustering est activé ?](#how-do-i-connect-to-my-cache-when-clustering-is-enabled)
* [Puis-je me connecter directement aux différentes partitions de mon cache ?](#can-i-directly-connect-to-the-individual-shards-of-my-cache)
* [Puis-je configurer le clustering pour un cache créé précédemment ?](#can-i-configure-clustering-for-a-previously-created-cache)
* [Puis-je configurer le clustering pour un cache De base ou Standard ?](#can-i-configure-clustering-for-a-basic-or-standard-cache)
* [Puis-je utiliser le clustering avec les fournisseurs d’état de session ASP.NET Redis et de mise en cache de la sortie ?](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)
* [J’obtiens des exceptions MOVE lorsque j’utilise StackExchange.Redis et le clustering, que dois-je faire ?](#i-am-getting-move-exceptions-when-using-stackexchangeredis-and-clustering-what-should-i-do)

### <a name="do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering"></a>Dois-je apporter des modifications à mon application cliente pour utiliser le clustering ?
* Lorsque le clustering est activé, seule la base de données 0 est disponible. Si votre application cliente utilise plusieurs bases de données et qu'elle essaie de lire ou d'écrire dans une base de données autre que 0, l'exception suivante est levée. `Unhandled Exception: StackExchange.Redis.RedisConnectionException: ProtocolFailure on GET --->` `StackExchange.Redis.RedisCommandException: Multiple databases are not supported on this server; cannot switch to database: 6`
  
  Pour plus d’informations, consultez [Redis Cluster Specification - Implemented subset](https://redis.io/topics/cluster-spec#implemented-subset)(Spécification de cluster Redis - Implémentation de jeu de données).
* Si vous utilisez [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/), vous devez utiliser la version 1.0.481 ou une version ultérieure. Vous vous connectez au cache à l’aide des [points de terminaison, ports et clés](cache-configure.md#properties) que vous utilisez pour vous connecter à un cache pour lequel le clustering n’est pas activé. La seule différence est que toutes les lectures et les écritures doivent être effectuées sur la base de données 0.
  
  D’autres clients peuvent avoir des exigences différentes. Consultez [Tous les clients Redis prennent-ils en charge le clustering ?](#do-all-redis-clients-support-clustering)
* Si votre application utilise plusieurs opérations sur les clés traitées par lot dans une seule commande, toutes les clés doivent se trouver dans la même partition. Pour trouver les clés dans la même partition, consultez [Comment les clés sont-elles distribuées dans un cluster ?](#how-are-keys-distributed-in-a-cluster)
* Si vous utilisez un fournisseur d’état de session ASP.NET Redis, vous devez utiliser la version 2.0.1 ou une version ultérieure. Consultez [Puis-je utiliser le clustering avec les fournisseurs d’état de session ASP.NET Redis et de mise en cache de la sortie ?](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)

### <a name="how-are-keys-distributed-in-a-cluster"></a>Comment les clés sont-elles distribuées dans un cluster ?
Selon la documentation Redis [Modèle de distribution de clés](https://redis.io/topics/cluster-spec#keys-distribution-model) : l’espace de clé est fractionné en 16 384 emplacements. Chaque clé est hachée et affectée à l’un de ces emplacements, qui sont répartis entre les nœuds du cluster. Vous pouvez configurer la partie de la clé qui est hachée pour vous assurer que plusieurs clés se trouvent dans la même partition à l’aide de balises de hachage.

* Clés avec une balise de hachage : si une partie de la clé est placée entre `{` et `}`, seule cette partie de la clé est hachée aux fins de détermination de l'emplacement de hachage d'une clé. Par exemple, les 3 clés suivantes se trouvent dans la même partition : `{key}1`, `{key}2` et `{key}3`, étant donné que seule la partie `key` du nom est hachée. Pour obtenir une liste complète des spécifications de balises de hachage de clés, consultez [Balises de hachage de clés](https://redis.io/topics/cluster-spec#keys-hash-tags).
* Clés sans balise de hachage : le nom entier de la clé est utilisé pour le hachage. Il en résulte une distribution statistiquement uniforme sur les partitions du cache.

Pour optimiser les performances et le débit, nous vous recommandons de distribuer les clés uniformément. Si vous utilisez des clés avec une balise de hachage, il incombe à l’application de vérifier que les clés sont réparties uniformément.

Pour plus d’informations, consultez [Modèle de distribution de clés](https://redis.io/topics/cluster-spec#keys-distribution-model), [Partitionnement de données de cluster Redis](https://redis.io/topics/cluster-tutorial#redis-cluster-data-sharding) et [Balises de hachage de clés](https://redis.io/topics/cluster-spec#keys-hash-tags).

Pour accéder à un exemple de code relatif à l’utilisation du clustering et la localisation des clés dans une même partition avec le client StackExchange.Redis, consultez la partie [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) de l’exemple [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).

### <a name="what-is-the-largest-cache-size-i-can-create"></a>Quelle est la taille de cache la plus grande que je peux créer ?
La plus grande taille de cache Premium est 120 Go. Vous pouvez créer jusqu’à 10 partitions, ce qui donne une taille maximale de 1,2 To. Si vous avez besoin d’une plus grande taille, vous pouvez en [faire la demande](mailto:wapteams@microsoft.com?subject=Redis%20Cache%20quota%20increase). Pour plus d’informations, consultez [Tarification du Cache Azure pour Redis](https://azure.microsoft.com/pricing/details/cache/).

### <a name="do-all-redis-clients-support-clustering"></a>Tous les clients Redis prennent-ils en charge le clustering ?
Les clients ne prennent pas tous en charge le clustering Redis ! Consultez la documentation de votre bibliothèque pour vérifier que vous utilisez une bibliothèque et une version qui prennent en charge le clustering. StackExchange.Redis est une bibliothèque dont les récentes versions prennent en charge le clustering. Pour plus d’informations sur d’autres clients, consultez la section [Playing with the cluster](https://redis.io/topics/cluster-tutorial#playing-with-the-cluster) (Manipulation du cluster) du [didacticiel sur le cluster Redis](https://redis.io/topics/cluster-tutorial). 

Le protocole de clustering Redis nécessite que chaque client se connecte directement à chaque partition en mode clustering et qu’il définisse de nouvelles réponses d’erreur telles que « MOVED » et « CROSSSLOTS ». Toute tentative d’utilisation d’un client qui ne prend pas en charge le clustering avec un cache en mode cluster peut entraîner un grand nombre d’[exceptions de redirection MOVED](https://redis.io/topics/cluster-spec#moved-redirection) ou tout simplement endommager votre application si vous effectuez des demandes à plusieurs clés entre emplacements.

> [!NOTE]
> Si vous utilisez StackExchange.Redis comme client, assurez-vous d'utiliser la dernière version de [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/) 1.0.481 ou une version ultérieure pour que le clustering fonctionne correctement. Si vous rencontrez des problèmes avec les exceptions MOVE, consultez la [section consacrée aux exceptions MOVE](#move-exceptions) pour plus d’informations.
>

### <a name="how-do-i-connect-to-my-cache-when-clustering-is-enabled"></a>Comment puis-je me connecter à mon cache quand le clustering est activé ?
Vous pouvez vous connecter à votre cache à l’aide des [points de terminaison](cache-configure.md#properties), [ports](cache-configure.md#properties) et [clés](cache-configure.md#access-keys) que vous utilisez pour vous connecter à un cache pour lequel le clustering n’est pas activé. Redis gère le clustering sur le serveur principal pour que vous n’ayez pas à le gérer à partir de votre client.

### <a name="can-i-directly-connect-to-the-individual-shards-of-my-cache"></a>Puis-je me connecter directement aux différentes partitions de mon cache ?
Le protocole de clustering requiert que le client établisse les connexions de partition appropriée. Par conséquent, le client doit faire cela correctement pour vous. Ceci étant dit, chaque partition composée d’une paire de caches principal/réplica désignés collectivement sous le nom d’« instance de cache ». Vous pouvez vous connecter à ces instances de cache à l'aide de l'utilitaire redis-cli dans la branche [unstable](https://redis.io/download) du référentiel Redis sur GitHub. Cette version implémente la prise en charge de base lorsqu’elle est démarrée avec le commutateur `-c` . Pour plus d’informations, consultez [Playing with the cluster](https://redis.io/topics/cluster-tutorial#playing-with-the-cluster) sur [https://redis.io](https://redis.io) dans le [tutoriel concernant les clusters Redis](https://redis.io/topics/cluster-tutorial).

Sans TLS, utilisez les commandes suivantes.

```bash
Redis-cli.exe –h <<cachename>> -p 13000 (to connect to instance 0)
Redis-cli.exe –h <<cachename>> -p 13001 (to connect to instance 1)
Redis-cli.exe –h <<cachename>> -p 13002 (to connect to instance 2)
...
Redis-cli.exe –h <<cachename>> -p 1300N (to connect to instance N)
```

Avec TLS, remplacez `1300N` par `1500N`.

### <a name="can-i-configure-clustering-for-a-previously-created-cache"></a>Puis-je configurer le clustering pour un cache créé précédemment ?
Oui. Vérifiez d’abord que votre cache est de niveau Premium (mettez-le à l’échelle si ce n’est pas le cas). Vous devriez alors voir les options de configuration du cluster, en particulier une option permettant d’activer un cluster. Vous pouvez changer la taille du cluster une fois le cache créé ou après l’activation initiale du clustering.

   >[!IMPORTANT]
   >Vous ne pouvez pas annuler l’activation du clustering. Un cache avec clustering activé et une seule partition se comporte *différemment* d’un cache de la même taille mais *sans* clustering.

### <a name="can-i-configure-clustering-for-a-basic-or-standard-cache"></a>Puis-je configurer le clustering pour un cache De base ou Standard ?
Le clustering est disponible uniquement pour les caches de niveau Premium.

### <a name="can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers"></a>Puis-je utiliser le clustering avec les fournisseurs d’état de session ASP.NET Redis et de mise en cache de la sortie ?
* **Fournisseur de caches de sortie Redis** : aucune modification requise.
* **Fournisseur d’état de session Redis** : pour utiliser le clustering, vous devez utiliser [RedisSessionStateProvider](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) version 2.0.1 ou ultérieure, sans quoi une exception est levée. Il s’agit d’un changement cassant. Pour plus d’informations, consultez [Détails de la modification avec rupture pour la version 2.0.0](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details).

<a name="move-exceptions"></a>

### <a name="i-am-getting-move-exceptions-when-using-stackexchangeredis-and-clustering-what-should-i-do"></a>J’obtiens des exceptions MOVE lorsque j’utilise StackExchange.Redis et le clustering, que dois-je faire ?
Si vous utilisez StackExchange.Redis et recevez des exceptions `MOVE` lors du clustering, assurez-vous que vous utilisez [StackExchange.Redis 1.1.603](https://www.nuget.org/packages/StackExchange.Redis/) ou version ultérieure. Pour obtenir des instructions sur la configuration de vos applications .NET afin d’utiliser StackExchange.Redis, consultez [Configuration des clients de cache](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients).

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les fonctionnalités d’Azure Cache pour Redis.

* [Niveaux de service Premium Azure Cache pour Redis](cache-overview.md#service-tiers)

<!-- IMAGES -->

[redis-cache-clustering]: ./media/cache-how-to-premium-clustering/redis-cache-clustering.png

[redis-cache-clustering-selected]: ./media/cache-how-to-premium-clustering/redis-cache-clustering-selected.png

[redis-cache-redis-cluster-size]: ./media/cache-how-to-premium-clustering/redis-cache-redis-cluster-size.png
