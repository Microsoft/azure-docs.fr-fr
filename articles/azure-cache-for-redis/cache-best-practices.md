---
title: Bonnes pratiques en matière d’utilisation du cache Azure pour Redis
description: Découvrez comment utiliser efficacement le cache Azure pour Redis en suivant ces bonnes pratiques.
author: joncole
ms.service: cache
ms.topic: conceptual
ms.date: 01/06/2020
ms.author: joncole
ms.openlocfilehash: 1b62777ec647efc6d5aded573e681cadd6475b47
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654793"
---
# <a name="best-practices-for-azure-cache-for-redis"></a>Bonnes pratiques en matière d’utilisation du cache Azure pour Redis 
En suivant ces bonnes pratiques, vous pouvez optimiser les performances et la rentabilité d’utilisation de votre instance du cache Azure pour Redis.

## <a name="configuration-and-concepts"></a>Configuration et concepts
 * **Utilisez le niveau Standard ou Premium pour les systèmes de production.**  Le niveau De base est un système à nœud unique, sans réplication des données ni contrat SLA. En outre, utilisez au moins un cache C1.  Les caches C0 sont destinés aux scénarios de développement/test simples, car ils ont un cœur de processeur partagé, disposent de peu de mémoire et sont susceptibles de rencontrer des problèmes de nuisances d’instances voisines (effet de « noisy neighbor »).

 * **N’oubliez pas que Redis est un magasin de données en mémoire.**  [Cet article](cache-troubleshoot-data-loss.md) présente quelques scénarios où une perte de données est possible.

 * **Développez votre système de sorte qu’il puisse gérer les problèmes de connexion temporaires** [causés par les processus de mise à jour corrective et de reprise d’activité](cache-failover.md).

 * **Configurez le [paramètre maxmemory-reserved](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) afin d’améliorer la réactivité du système** dans des conditions de forte sollicitation de la mémoire.  Une valeur suffisante de réservation est particulièrement importante pour les charges de travail nécessitant beaucoup d’écritures, ou si vous stockez de gros volumes de données (100 Ko ou plus) dans Redis. Vous devez commencer par 10 % de la taille de votre cache, puis augmenter ce pourcentage si vous avez des charges lourdes en écritures.

 * **Redis fonctionne de manière optimale avec des valeurs plus petites**. Il est donc conseillé de fractionner les données plus volumineuses dans plusieurs clés.  Lisez avec attention les différents points exposés dans [cette discussion à propos de Redis](https://stackoverflow.com/questions/55517224/what-is-the-ideal-value-size-range-for-redis-is-100kb-too-large/).  Listez [cet article](cache-troubleshoot-client.md#large-request-or-response-size) illustrant un exemple de problème pouvant être provoqué par des valeurs élevées.

 * **Placez votre instance de cache et votre application dans la même région.**  La connexion à un cache situé dans une autre région peut considérablement augmenter la latence et diminuer la fiabilité.  Vous pouvez vous connecter en dehors d’Azure, mais cela est déconseillé, *en particulier si vous utilisez Redis comme cache*.  Si vous utilisez Redis simplement comme magasin de clés/valeurs, la latence n’est sans doute pas votre première préoccupation. 

 * **Réutilisez les connexions.**  La création de connexions supplémentaires est un processus coûteux qui augmente la latence. C’est pourquoi il est conseillé de réutiliser le plus souvent possible les connexions existantes. Si vous choisissez de créer d’autres connexions, assurez-vous de fermer les anciennes connexions avant de les libérer (même dans les langages de mémoire managée comme .NET ou Java).

 * **Configurez votre bibliothèque cliente pour qu’elle observe un *délai de connexion* d’au moins 15 secondes**, afin de laisser au système le temps de se connecter même dans des conditions de forte sollicitation du processeur.  La définition du délai de connexion à une valeur inférieure ne garantit pas que la connexion puisse être établie dans ce laps de temps.  Si un problème se produit (forte utilisation du processeur du client ou du serveur, par exemple), un délai de connexion trop court fait échouer la tentative de connexion. Ce comportement aggrave souvent une situation déjà détériorée.  Au lieu d’améliorer la situation, la diminution des délais d’attente aggrave le problème en forçant le système à redémarrer le processus de tentative de reconnexion, avec le risque de générer au final une boucle *connexion -> échec -> nouvelle tentative*. Nous conseillons généralement de conserver un délai de connexion d’au moins 15 secondes. En effet, il est préférable d’attendre 15 ou 20 secondes et que la première tentative de connexion réussisse, plutôt que de faire plusieurs tentatives rapprochées qui échouent. Avec ce genre de boucle de nouvelles tentatives, votre système peut en fin de compte rester indisponible durant plus longtemps que si vous lui accordez initialement plus de temps.  
     > [!NOTE]
     > Ce conseil concerne la *tentative de connexion*, pas la durée d’attente souhaitée pour terminer une *opération* comme GET ou SET.

 * **Évitez les opérations coûteuses**. Certaines opérations Redis, comme la commande [KEYS](https://redis.io/commands/keys), sont *très* coûteuses et sont donc à proscrire.  Pour plus d’informations, lisez ces quelques considérations à propos des [commandes de longue durée](cache-troubleshoot-server.md#long-running-commands)

 * **Utilisez le chiffrement TLS**. Azure Cache pour Redis nécessite des communications chiffrées avec le protocole TLS par défaut.  Les versions 1.0, 1.1 et 1.2 de TLS sont actuellement prises en charge.  Toutefois, TLS 1.0 et 1.1 étant en passe de dépréciation dans l’ensemble du secteur, utilisez TLS 1.2 dans la mesure du possible.  Si votre outil ou bibliothèque de client ne prend pas en charge TLS, vous pouvez activer des connexions non chiffrées par le biais du [portail Azure](cache-configure.md#access-ports) ou des [API de gestion](/rest/api/redis/redis/update).  Dans les cas où il est impossible d’établir des connexions chiffrées, nous vous recommandons de placer votre cache et votre application cliente dans un réseau virtuel.  Pour plus d’informations sur les ports utilisés dans le scénario de mise en cache du réseau virtuel, consultez [ce tableau](cache-how-to-premium-vnet.md#outbound-port-requirements).

 * **Délai d’inactivité**-les inconnus Azure ont actuellement un délai d’expiration de 10 minutes pour les connexions, donc cette valeur doit être inférieure à 10 minutes.

## <a name="memory-management"></a>Gestion de la mémoire
Il y a plusieurs points à prendre en compte en ce qui concerne l’utilisation de la mémoire dans votre instance de serveur Redis.  En voici quelques-uns :

 * **Choisissez une [stratégie d’éviction](https://redis.io/topics/lru-cache) appropriée pour votre application.**  La stratégie par défaut pour Azure Redis est *volatile-lru*, qui spécifie que seules les clés dont la durée de vie a été définie peuvent être supprimées.  Si aucune clé n’a une valeur de durée de vie, le système ne supprime pas de clés.  Si vous souhaitez que le système autorise la suppression de toutes les clés dans des conditions de forte sollicitation de la mémoire, choisissez plutôt la stratégie *allkeys-lru*.

 * **Définissez le délai d’expiration de vos clés.**  L’expiration va entraîner la suppression des clés de manière proactive, et non uniquement en cas de forte sollicitation de la mémoire.  Quand une opération de suppression a lieu en raison d’une forte sollicitation de la mémoire, cela peut entraîner une charge supplémentaire sur votre serveur.  Pour plus d’informations, consultez la documentation concernant les commandes [EXPIRE](https://redis.io/commands/expire) et [EXPIREAT](https://redis.io/commands/expireat).

## <a name="client-library-specific-guidance"></a>Conseils pour la bibliothèque cliente
 * [StackExchange.Redis (.NET)](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-stackexchange-redis-md)
 * [Java - Quel client dois-je utiliser ?](https://gist.github.com/warrenzhu25/1beb02a09b6afd41dff2c27c53918ce7#file-azure-redis-java-best-practices-md)
 * [Lettuce (Java)](https://gist.github.com/warrenzhu25/181ccac7fa70411f7eb72aff23aa8a6a#file-azure-redis-lettuce-best-practices-md)
 * [Jedis (Java)](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-java-jedis-md)
 * [Node.JS](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-node-js-md)
 * [PHP](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-php-md)
 * [Fournisseur d’état de session ASP.NET](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f#file-redis-bestpractices-session-state-provider-md)


## <a name="when-is-it-safe-to-retry"></a>Quand faut-il retenter des opérations ?
Malheureusement, il n’existe pas de réponse simple.  Chaque application doit déterminer quelles opérations peuvent être retentées ou pas.  Chaque opération a ses exigences propres et présente des dépendances interclés différentes.  Voici quelques éléments de réflexion :

 * Il arrive d’observer des erreurs côté client même si Redis a correctement exécuté la commande que vous lui avez demandée.  Par exemple :
    - Les délais d’expiration sont un concept côté client.  Si l’opération parvient au serveur, celui-ci exécute la commande même si le client a entre-temps abandonné l’opération en attente.  
    - Quand une erreur se produit sur la connexion de socket, il n’est pas possible de savoir si l’opération a pu être exécutée sur le serveur.  Par exemple, l’erreur de connexion peut se produire dans le laps de temps entre le traitement de la requête par le serveur et la réception de la réponse par le client.
 * Comment mon application réagit-elle si j’exécute accidentellement deux fois la même opération ?  Par exemple, que se passe-t-il si j’incrémente un entier deux fois au lieu d’une ?  Mon application écrit-elle sur la même clé à partir d’emplacements différents ?  Que se passe-t-il si la logique de nouvelle tentative remplace une valeur définie dans une autre partie de mon application ?

Si vous souhaitez tester le comportement de votre code dans des conditions d’erreur, utilisez la [fonctionnalité Redémarrer](cache-administration.md#reboot). Le redémarrage vous permet de voir l’impact des problèmes de connexion temporaires sur votre application.

## <a name="performance-testing"></a>Tests de performances
 * **Commencez par utiliser `redis-benchmark.exe`** pour avoir une idée du débit ou de la latence possible avant d’écrire vos propres tests de performances.  La documentation de redis-benchmark est [disponible ici](https://redis.io/topics/benchmarks).  Notez que redis-benchmark ne prenant pas en charge TLS, vous devez donc [activer le port non-TLS via le portail](cache-configure.md#access-ports) avant de lancer le test.  [Une version de redis-benchmark.exe pour Windows est disponible ici](https://github.com/MSOpenTech/redis/releases)
 * La machine virtuelle cliente utilisée pour le test doit se trouver **dans la même région** que votre instance de cache Redis.
 * **Nous vous conseillons d’utiliser des machines virtuelles Dv2** pour votre client, car elles sont matériellement plus performantes et offrent de meilleurs résultats.
 * Assurez-vous que la machine virtuelle cliente que vous utilisez **possède au moins autant de puissance de calcul et de bande passante* que le cache testé. 
 * **Activez VRSS** sur la machine cliente si vous êtes sur Windows.  [Vous trouverez plus d’informations ici](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn383582(v=ws.11)).  Exemple de script PowerShell :
     >PowerShell -ExecutionPolicy Unrestricted Enable-NetAdapterRSS -Name (    Get-NetAdapter).Name 

 * **Envisagez d’utiliser des instances Redis de niveau Premium**.  Ces tailles de cache offrent une meilleure latence de réseau et un débit supérieur, car les instances s’exécutent sur un matériel plus performant pour le processeur et le réseau.

     > [!NOTE]
     > Les résultats observés sur les performances sont [publiés ici](cache-planning-faq.md#azure-cache-for-redis-performance) à titre de référence.   Par ailleurs, sachez que SSL/TLS augmente la charge, et que vous pouvez donc constater des latences et/ou des débits différents si vous utilisez le chiffrement de transport.

### <a name="redis-benchmark-examples"></a>Exemples redis-benchmark
**Configuration des prétests** : préparez l’instance de cache avec les données requises pour les commandes de test de la latence et du débit ci-dessous.
> redis-benchmark -h yourcache.redis.cache.windows.net -a yourAccesskey -t SET -n 10 -d 1024 

**Pour tester la latence :** testez les requêtes GET avec une charge utile de 1 Ko.
> redis-benchmark -h yourcache.redis.cache.windows.net -a yourAccesskey -t GET -d 1024 -P 50 -c 4

**Pour tester le débit :** requêtes GET en pipeline avec une charge utile de 1 Ko.
> redis-benchmark -h yourcache.redis.cache.windows.net -a yourAccesskey -t GET -n 1000000 -d 1024 -P 50 -c 50