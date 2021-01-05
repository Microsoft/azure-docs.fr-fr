---
title: Résoudre les problèmes côté client liés à Azure Cache pour Redis
description: Découvrez comment résoudre les problèmes courants côté client liés à Azure Cache pour Redis, comme les sollicitations de la mémoire du client, les pics de trafic, l’utilisation élevée du processeur, les limites de bande passante, les demandes volumineuses ou les réponses de grande taille.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: troubleshooting
ms.date: 10/18/2019
ms.openlocfilehash: 122c96c95aea794fbba9cab8a9a5b867f9f34b48
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88008965"
---
# <a name="troubleshoot-azure-cache-for-redis-client-side-issues"></a>Résoudre les problèmes côté client liés à Azure Cache pour Redis

Cette section traite de la résolution des problèmes qui se produisent à la suite d’une condition sur le client Redis utilisé par votre application.

- [Saturation de la mémoire sur le client Redis](#memory-pressure-on-redis-client)
- [Augmentation de trafic](#traffic-burst)
- [Utilisation importante du processeur du client](#high-client-cpu-usage)
- [Limitation de la bande passante côté client](#client-side-bandwidth-limitation)
- [Taille importante de la demande ou de la réponse](#large-request-or-response-size)

## <a name="memory-pressure-on-redis-client"></a>Sollicitation de la mémoire sur le client Redis

La sollicitation de la mémoire sur l’ordinateur client génère toutes sortes de problèmes de performances susceptibles de retarder le traitement des réponses provenant du cache. Lorsque la mémoire est trop sollicitée, le système peut paginer les données vers le disque. Cette _pagination_ ralentit considérablement le système.

Pour détecter une sollicitation de la mémoire sur le client :

- Supervisez l’utilisation de la mémoire sur l’ordinateur et vérifiez qu’elle ne dépasse pas la mémoire disponible.
- Supervisez le compteur de performances `Page Faults/Sec` du client. Lors d’un fonctionnement normal, la plupart des systèmes peuvent rencontrer des défauts de page. Les pics de défauts de page correspondant à des délais d’attente de requêtes peuvent indiquer une sollicitation de la mémoire.

Vous pouvez réduire la sursollicitation de la mémoire sur le client de plusieurs façons :

- Analysez vos modèles d’utilisation de la mémoire pour réduire la consommation de mémoire sur le client.
- Mettez à niveau votre machine virtuelle cliente pour bénéficier de davantage de mémoire.

## <a name="traffic-burst"></a>Augmentation de trafic

Les augmentations de trafic combinées à des paramètres `ThreadPool` insatisfaisants peuvent retarder le traitement des données déjà envoyées par le serveur Redis, mais pas encore consommées côté client.

Supervisez la façon dont évoluent vos statistiques `ThreadPool` au fil du temps à l’aide d’un [exemple `ThreadPoolLogger`](https://github.com/JonCole/SampleCode/blob/master/ThreadPoolMonitor/ThreadPoolLogger.cs). Pour une analyse plus approfondie, vous pouvez utiliser les messages `TimeoutException` de StackExchange.Redis comme ci-dessous :

```output
    System.TimeoutException: Timeout performing EVAL, inst: 8, mgr: Inactive, queue: 0, qu: 0, qs: 0, qc: 0, wr: 0, wq: 0, in: 64221, ar: 0,
    IOCP: (Busy=6,Free=999,Min=2,Max=1000), WORKER: (Busy=7,Free=8184,Min=2,Max=8191)
```

Dans l’exception précédente, plusieurs problèmes sont intéressants :

- Notez que dans les sections `IOCP` et `WORKER`, la valeur de `Busy` est supérieure à la valeur de `Min`. Cette différence signifie que vos paramètres `ThreadPool` ont besoin d’être ajustés.
- Vous pouvez également voir `in: 64221`. Cette valeur indique que 64 211 octets ont été reçus au niveau de la couche de socket du noyau du client, mais qu’ils n’ont pas encore été lus par l’application. En général, cette différence signifie que votre application (par exemple, StackExchange.Redis) ne lit pas les données du réseau aussi rapidement que le serveur les lui envoie.

Vous pouvez [configurer vos paramètres `ThreadPool`](cache-management-faq.md#important-details-about-threadpool-growth) pour que votre pool de threads puisse rapidement faire l’objet d’un scale-up en cas d’augmentation du trafic.

## <a name="high-client-cpu-usage"></a>Utilisation importante du processeur du client

Une utilisation élevée du processeur du client indique que le système ne peut pas exécuter la charge de travail qui lui a été ajoutée dans un délai normal. Même si le cache a envoyé la réponse rapidement, le client ne peut pas traiter la réponse en temps voulu.

Supervisez l’utilisation du processeur du client à l’échelle du système à l’aide des métriques qui sont disponibles dans le portail Azure ou via les compteurs de performances de l’ordinateur. Ne tenez pas compte du processeur de *processus*, car le processeur peut très bien être faiblement utilisé par un processus tout en étant fortement sollicité à l’échelle du système. Regardez les pics d’utilisation du processeur qui correspondent à des délais d’expiration. Quand le processeur est très utilisé, vous pouvez également voir s’afficher des valeurs `in: XXX` élevées dans les messages d’erreur `TimeoutException`, comme ceux décrits dans la section [Augmentation de trafic](#traffic-burst).

> [!NOTE]
> StackExchange.Redis 1.1.603 et versions ultérieures inclut la mesure `local-cpu` dans les messages d’erreur `TimeoutException`. Vérifiez que vous utilisez la dernière version en date du [package NuGet StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/). Nous corrigeons en permanence le code pour le rendre plus robuste aux délais d’expiration. Il est donc primordial d’utiliser la dernière version.
>

Pour remédier à une utilisation élevée du processeur d’un client :

- Recherchez la cause des pics d’utilisation du processeur.
- Mettez à niveau votre machine virtuelle cliente pour bénéficier d’une plus grande capacité de processeur.

## <a name="client-side-bandwidth-limitation"></a>Limitation de la bande passante côté client

Selon l’architecture des ordinateurs clients, ces ordinateurs peuvent afficher des limites concernant la bande passante réseau qu’ils peuvent utiliser. Si le client dépasse la bande passante disponible en surchargeant la capacité du réseau, les données ne sont pas traitées côté client aussi rapidement que le serveur les envoie. Cette situation peut entraîner des délais d’attente.

Supervisez la façon dont votre utilisation de la bande passante évolue au fil du temps, à l’aide d’un [exemple `BandwidthLogger`](https://github.com/JonCole/SampleCode/blob/master/BandWidthMonitor/BandwidthLogger.cs). Ce code peut ne pas s’exécuter correctement dans certains environnements aux autorisations restreintes (tels que les sites web Azure).

Réduisez l’utilisation de la bande passante réseau ou augmentez la taille de la machine virtuelle cliente pour bénéficier d’une plus grande capacité réseau.

## <a name="large-request-or-response-size"></a>Taille importante de la demande ou de la réponse

Une demande/réponse volumineuse peut entraîner des délais d’expiration. Par exemple, supposons que la durée du délai d’expiration que vous avez configurée sur votre client soit de 1 seconde. Votre application demande deux clés (par exemple, « A » et « B ») en même temps (à l’aide de la même connexion réseau physique). La plupart des clients prennent en charge le traitement en pipeline des requêtes, de sorte que les deux requêtes A et B sont envoyées l’une après l’autre sans attendre les réponses. Le serveur renvoie les réponses dans le même ordre. Si la réponse A est volumineuse, elle peut consommer la majeure partie du délai d’expiration des requêtes suivantes.

Dans l’exemple suivant, les requêtes A et B sont envoyées rapidement au serveur. Le serveur commence rapidement à envoyer les réponses A et B. En raison des temps de transfert de données, la réponse B doit attendre l’expiration de la réponse A, même si le serveur a répondu rapidement.

```console
|-------- 1 Second Timeout (A)----------|
|-Request A-|
     |-------- 1 Second Timeout (B) ----------|
     |-Request B-|
            |- Read Response A --------|
                                       |- Read Response B-| (**TIMEOUT**)
```

Cette demande/réponse est difficile à mesurer. Vous pouvez instrumenter votre code client pour suivre les requêtes et les réponses volumineuses.

Les solutions possibles pour la gestion des réponses volumineuses sont variées, et incluent notamment :

1. Optimiser votre application pour prendre en charge un grand nombre de petites valeurs plutôt qu’un petit nombre de grandes valeurs
    - La meilleure solution consiste à diviser vos données en valeurs plus petites.
    - Lisez le billet [What is the ideal value size range for redis? Is 100 KB too large?](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ) pour savoir pourquoi il est recommandé d’utiliser des valeurs moins élevées.
1. Augmenter la taille de votre machine virtuelle pour obtenir des capacités de bande passante plus élevées
    - Une plus grande quantité de bande passante sur votre machine virtuelle cliente ou serveur peut réduire le temps de transfert des données pour les réponses volumineuses.
    - Comparez l’utilisation actuelle du réseau par les deux ordinateurs aux limites associées à la taille de votre machine virtuelle. L’augmentation de la bande passante sur le serveur ou le client uniquement peut ne pas suffire.
1. Augmenter le nombre d’objets de connexion qu’utilise votre application
    - Utilisez un tourniquet (round-robin) pour envoyer des requêtes vers différents objets de connexion.

## <a name="additional-information"></a>Informations supplémentaires

- [Résoudre les problèmes côté serveur liés à Azure Cache pour Redis](cache-troubleshoot-server.md)
- [Comment puis-je évaluer et tester les performances de mon cache ?](cache-management-faq.md#how-can-i-benchmark-and-test-the-performance-of-my-cache)
