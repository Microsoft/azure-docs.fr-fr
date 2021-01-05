---
title: Check-list des performances et de la scalabilité pour le stockage Table - Stockage Azure
description: Check-list de pratiques validées pour le stockage Table en vue du développement d’applications performantes.
services: storage
author: tamram
ms.author: tamram
ms.service: storage
ms.topic: overview
ms.date: 10/10/2019
ms.subservice: tables
ms.custom: devx-track-csharp
ms.openlocfilehash: 71b1f3cfa1df86b417c468d56f67cd7fe8d71d73
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004702"
---
# <a name="performance-and-scalability-checklist-for-table-storage"></a>Check-list des performances et de la scalabilité pour le stockage Table

Microsoft a mis au point un certain nombre de pratiques qui ont fait leurs preuves pour le développement d’applications hautes performances avec le stockage Table. Cette check-list fournit des pratiques clés que les développeurs peuvent utiliser pour optimiser les performances. Gardez ces pratiques à l’esprit lorsque vous concevez votre application et tout au long du processus de développement.

Le stockage Azure a des objectifs de scalabilité et de performances pour la capacité, le taux de transactions et la bande passante. Pour plus d’informations sur les cibles de scalabilité de Stockage Azure, consultez [Cibles de scalabilité et de performances pour les comptes de stockage standard](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2ftables%2ftoc.json) et [Cibles de scalabilité et de performances pour le stockage Table](scalability-targets.md).

## <a name="checklist"></a>Liste de contrôle

Cet article organise les pratiques validées pour les performances dans une check-list que vous pouvez suivre lors du développement de votre application de stockage Table.

| Terminé | Category | Considérations relatives à la conception |
| --- | --- | --- |
| &nbsp; |Objectifs de scalabilité |[Pouvez-vous concevoir votre application pour ne pas qu’elle utilise plus que le nombre maximal de comptes de stockage ?](#maximum-number-of-storage-accounts) |
| &nbsp; |Objectifs de scalabilité |[Cherchez-vous à ne pas vous approcher des limites de capacité et de transactions ?](#capacity-and-transaction-targets) |
| &nbsp; |Objectifs de scalabilité |[Vous approchez-vous des objectifs d'évolutivité en termes d'entités par seconde ?](#targets-for-data-operations) |
| &nbsp; |Mise en réseau |[Les appareils côté client disposent-ils d’une bande passante suffisamment large et d’une latence suffisamment faible pour parvenir aux performances nécessaires ?](#throughput) |
| &nbsp; |Mise en réseau |[La qualité du lien réseau côté client est-elle suffisamment élevée ?](#link-quality) |
| &nbsp; |Mise en réseau |[L’application cliente se trouve-t-elle dans la même région que le compte de stockage ?](#location) |
| &nbsp; |Accès direct au client |[Utilisez-vous des signatures d’accès partagé (SAP) et un partage des ressources cross-origin (CORS) pour permettre l’accès direct au stockage Azure ?](#sas-and-cors) |
| &nbsp; |Traitement par lot |[Votre application traite-t-elle les mises à jour par lot à l’aide de transactions de groupe d’entités ?](#batch-transactions) |
| &nbsp; |Configuration .NET |[Utilisez-vous .NET Core 2.1 ou ultérieur pour obtenir des performances optimales ?](#use-net-core) |
| &nbsp; |Configuration .NET |[Avez-vous configuré votre client pour qu'il utilise un nombre suffisant de connexions simultanées ?](#increase-default-connection-limit) |
| &nbsp; |Configuration .NET |[Pour les applications .NET, avez-vous configuré .NET pour qu’il utilise un nombre suffisant de threads ?](#increase-minimum-number-of-threads) |
| &nbsp; |Parallélisme |[Avez-vous vérifié que le parallélisme est limité de façon appropriée, de manière à ne pas surcharger les fonctionnalités de votre client ni s’approcher des objectifs de scalabilité ?](#unbounded-parallelism) |
| &nbsp; |Outils |[Utilisez-vous la version la plus récente des outils et bibliothèques clientes fournis par Microsoft ?](#client-libraries-and-tools) |
| &nbsp; |Nouvelle tentatives |[Utilisez-vous une stratégie de nouvelles tentatives avec backoff exponentiel pour les erreurs de limitation et les délais d’expiration ?](#timeout-and-server-busy-errors) |
| &nbsp; |Nouvelle tentatives |[Votre application empêche-t-elle les nouvelles tentatives pour les erreurs non renouvelables ?](#non-retryable-errors) |
| &nbsp; |Configuration |[Utilisez-vous JSON pour vos demandes de table ?](#use-json) |
| &nbsp; |Configuration |[Avez-vous désactivé l’algorithme Nagle pour améliorer les performances des petites requêtes ?](#disable-nagle) |
| &nbsp; |Tables et partitions |[Avez-vous correctement partitionné vos données ?](#schema) |
| &nbsp; |Partitions actives |[Évitez-vous les modèles « Ajouter après uniquement » ou « Ajouter avant uniquement » ?](#append-only-and-prepend-only-patterns) |
| &nbsp; |Partitions actives |[Vos opérations d'insertion/de mise à jour couvrent-elles plusieurs partitions ?](#high-traffic-data) |
| &nbsp; |Étendue de requête |[Avez-vous conçu votre schéma pour qu'il autorise l'utilisation des requêtes de point dans la plupart des cas et l'utilisation modérée des requêtes de tables ?](#query-scope) |
| &nbsp; |Densité des requêtes |[En règle générale, vos requêtes n'analysent et ne renvoient-elles que les lignes qui seront utilisées par votre application ?](#query-density) |
| &nbsp; |Limitation des données retournées |[Utilisez-vous le filtrage pour éviter le renvoi d'entités inutiles ?](#limiting-the-amount-of-data-returned) |
| &nbsp; |Limitation des données retournées |[Utilisez-vous la projection pour éviter le renvoi de propriétés inutiles ?](#limiting-the-amount-of-data-returned) |
| &nbsp; |Dénormalisation |[Avez-vous dénormalisé vos données de manière à éviter les requêtes inefficaces ou les demandes de lecture multiples lorsque vous essayez de récupérer des données ?](#denormalization) |
| &nbsp; |Insérer, mettre à jour et supprimer |[Effectuez-vous un traitement par lots des demandes qui doivent être transactionnelles ou qui peuvent être effectuées en même temps pour réduire les allers-retours ?](#batching) |
| &nbsp; |Insérer, mettre à jour et supprimer |[Évitez-vous de récupérer une entité pour déterminer simplement s'il faut appeler l'opération insert ou update ?](#upsert) |
| &nbsp; |Insérer, mettre à jour et supprimer |[Avez-vous envisagé de stocker des séries de données qui seront fréquemment récupérées ensemble dans une seule entité sous la forme de propriétés plutôt que d'entités multiples ?](#storing-data-series-in-a-single-entity) |
| &nbsp; |Insérer, mettre à jour et supprimer |[Dans le cas des tables qui sont toujours récupérées ensemble et qui peuvent être écrites par lots (des données de séries temporelles, par exemple), avez-vous envisagé d’utiliser des objets blob à la place de tables ?](#storing-structured-data-in-blobs) |

## <a name="scalability-targets"></a>Objectifs de scalabilité

Si votre application s’approche de l’un des objectifs d’extensibilité, voire le dépasse, une limitation ou des latences de transaction accrues peuvent survenir. Lorsque le stockage Azure limite votre application, le service commence à retourner les codes d’erreur 503 (Serveur occupé) ou 500 (Délai d’expiration de l’opération). Pour améliorer les performances de votre application, il est important d’éviter de telles erreurs en restant dans les limites des objectifs de scalabilité.

Pour plus d’informations sur les objectifs de scalabilité concernant le service de Table, consultez [Objectifs de scalabilité et de performances pour le stockage de tables](scalability-targets.md).

### <a name="maximum-number-of-storage-accounts"></a>Nombre maximal de comptes de stockage

Si vous atteignez le nombre maximal de comptes de stockage autorisés pour une combinaison abonnement-région, utilisez-vous plusieurs comptes de stockage afin d’effectuer un partitionnement en vue d’augmenter les entrées, les sorties, les opérations d’E/S par seconde (IOPS) ou la capacité ? Dans ce scénario, Microsoft recommande, si possible, de tirer parti de l’augmentation du nombre maximal de comptes de stockage, afin de réduire le nombre de comptes de stockage nécessaires pour votre charge de travail. Contactez le [support Azure](https://azure.microsoft.com/support/options/) pour demander l’augmentation des limites de votre compte de stockage. Pour plus d’informations, consultez [Annonce de comptes de stockage plus grands et à plus grande échelle](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/).

### <a name="capacity-and-transaction-targets"></a>Objectifs de capacité et de transaction

Si votre application s’approche des objectifs d’extensibilité d’un seul compte de stockage, pensez à appliquer l’une des méthodes suivantes :  

- Réexaminez la charge de travail à cause de laquelle votre application s’approche de l’objectif d’extensibilité ou le dépasse. Est-il possible de la concevoir différemment pour qu’elle utilise moins de bande passante ou de capacité, ou moins de transactions ?
- Si votre application doit dépasser l’un des objectifs de scalabilité, vous devez créer plusieurs comptes de stockage et partitionner vos données d’application sur ces comptes de stockage. Si vous optez pour ce modèle, veillez à concevoir votre application de telle sorte que vous puissiez ajouter davantage de comptes de stockage à l’avenir en vue de l’équilibrage de la charge. Aucun coût autre que l’utilisation (données stockées, transactions effectuées, données transférées, etc.) n’est associé à ces comptes de stockage.
- Si votre application s’approche des objectifs de bande passante, pensez à compresser les données côté client afin de réduire la bande passante nécessaire à l’envoi des données au stockage Azure.
    Même si la compression des données peut réduire la bande passante et améliorer les performances du réseau, elle présente des inconvénients. Évaluez l’impact sur les performances qu’entraînent les exigences de traitement supplémentaires liées à la compression et à la décompression des données côté client. Gardez à l’esprit que le stockage de données compressées peut compliquer la résolution des problèmes, car il peut être plus difficile de voir les données à l’aide d’outils standard.
- Si votre application s’approche des objectifs de scalabilité, vérifiez que vous utilisez bien un backoff exponentiel pour les nouvelles tentatives. Il est préférable d’éviter de s’approcher des objectifs de scalabilité en implémentant les recommandations fournies dans cet article. Toutefois, l’utilisation d’un backoff exponentiel pour les nouvelles tentatives va empêcher votre application d’effectuer une nouvelle tentative rapidement, ce qui pourrait compliquer la limitation. Pour plus d’informations, consultez la section intitulée [Erreurs d’expiration de délai et de serveur occupé](#timeout-and-server-busy-errors).

### <a name="targets-for-data-operations"></a>Cibles pour les opérations de données

Stockage Azure équilibre la charge à mesure que le trafic vers votre compte de stockage augmente, mais si le trafic présente des rafales soudaines, vous risquez de ne pas pouvoir récupérer ce volume de débit immédiatement. Attendez-vous à constater une limitation et/ou des délais d’expiration pendant la rafale, car Stockage Azure équilibre automatiquement la charge de votre table. Une accélération progressive procure généralement de meilleurs résultats, car le système a le temps d’équilibrer la charge de manière appropriée.

#### <a name="entities-per-second-storage-account"></a>Entités par seconde (compte)

S’agissant de l’accès aux tables, la limite de scalabilité est de 20 000 entités (d’une taille individuelle de 1 Ko) par seconde pour un compte. En règle générale, chaque entité insérée, mise à jour, supprimée ou analysée est comptabilisée. Une insertion par lots composée de 100 entités compte donc pour 100 entités. De même, une requête qui analyse 1 000 entités et en renvoie 5 compte pour 1 000 entités.

#### <a name="entities-per-second-partition"></a>Entités par seconde (partition)

Dans une partition unique, l’objectif de scalabilité pour l’accès aux tables est de 2 000 entités (d’une taille individuelle de 1 Ko) par seconde, en utilisant le même mode de calcul que celui décrit dans la section précédente.

## <a name="networking"></a>Mise en réseau

Les contraintes de réseau physiques de l’application peuvent avoir un impact majeur sur les performances. Les sections suivantes abordent certaines des limitations auxquelles les utilisateurs peuvent être confrontés.  

### <a name="client-network-capability"></a>Fonctionnalités réseau du client

La bande passante et la qualité du lien réseau jouent un rôle important dans les performances de l’application, comme l’expliquent les sections suivantes.

#### <a name="throughput"></a>Débit

Dans le cas de la bande passante, le problème est souvent dû aux capacités du client. Dans le cas des instances Azure plus importantes, les cartes réseau présentent une capacité supérieure. Si vous avez besoin de limites réseau plus élevées à partir d’un seul ordinateur, vous devez donc envisager d’utiliser une instance plus grande ou davantage de machines virtuelles. Si vous accédez au stockage Azure à partir d’une application locale, alors la même règle s’applique : comprendre les capacités réseau de l’appareil client et la connectivité réseau à l’emplacement de stockage Azure, et l’améliorer selon vos besoins ou concevoir votre application conformément à ces capacités.

#### <a name="link-quality"></a>Qualité du lien

Comme c’est le cas pour toute utilisation du réseau, n’oubliez pas que les conditions réseau qui génèrent des erreurs et une perte de paquets ralentissent le débit effectif.  L’utilisation de WireShark ou de NetMon peut vous aider à diagnostiquer ce problème.  

### <a name="location"></a>Emplacement

Dans un environnement distribué, le fait de placer le client à proximité du serveur se traduit par des performances optimales. Pour accéder à Azure Storage avec un minimum de latence, votre client doit idéalement se trouver dans la même région Azure. Par exemple, si vous disposez d’un site web Azure qui utilise le stockage Azure, tous deux doivent se trouver dans la même région (USA Ouest ou Asie Sud-Est, par exemple). Cela réduit à la fois la latence et les coûts, puisque l’utilisation de la bande passante dans une seule région était gratuite.  

Si des applications clientes accèdent au stockage Azure sans être hébergées dans Azure (c’est le cas, par exemple, des applications pour appareil mobile ou des services d’entreprise locaux), le fait de placer le compte de stockage dans une région proche de ces appareils peut réduire la latence. Si vos clients sont distribués à grande échelle (par exemple, certains en Amérique du Nord et d’autres en Europe), il est conseillé d’utiliser un compte de stockage pour chaque région. Cette méthode est plus facile à implémenter si les données stockées par l’application sont propres à certains utilisateurs, et elle ne nécessite pas de réplication des données entre différents comptes de stockage.

## <a name="sas-and-cors"></a>SAP et CORS

Supposons que vous deviez autoriser du code, tel que du code JavaScript qui s’exécute dans le navigateur web d’un utilisateur ou dans une application de téléphone mobile, à accéder aux données du stockage Azure. L’une des méthodes possibles consiste à créer une application de service qui serve de proxy. L’appareil de l’utilisateur s’authentifie auprès du service, qui à son tour autorise l’accès aux ressources du stockage Azure. Vous évitez ainsi d’exposer vos clés de compte de stockage sur des appareils non sécurisés. Cependant, cette méthode entraîne une surcharge importante pour l’application du service, dans la mesure où toutes les données transférées entre l’appareil de l’utilisateur et le stockage Azure doivent transiter par l’application du service.

Vous pouvez éviter d’utiliser une application de service en tant que proxy pour le stockage Azure en utilisant des signatures d’accès partagé (SAS). Avec les signatures d’accès partagé, vous pouvez autoriser l’appareil de votre utilisateur à adresser directement des requêtes au stockage Azure par le biais d’un jeton à accès limité. Par exemple, si un utilisateur souhaite charger une photo vers votre application, votre application de service peut générer une signature d’accès partagé et l’envoyer à l’appareil de l’utilisateur. Le jeton SAS peut accorder l’autorisation d’écrire dans une ressource du stockage Azure pendant un intervalle de temps spécifié, au bout duquel le jeton SAS expire. Pour plus d’informations sur les SAS, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAS)](../common/storage-sas-overview.md).  

En règle générale, un navigateur web n’autorise pas le code JavaScript d’une page hébergée par un site web d’un domaine à effectuer certaines opérations, telles que les opérations d’écriture, sur un autre domaine. Connue sous le nom de stratégie de même origine, cette stratégie empêche un script malveillant d’une page d’obtenir l’accès aux données d’une autre page web. Toutefois, la stratégie de même origine peut représenter un obstacle lorsque vous créez une solution dans le cloud. Le partage des ressources cross-origin (CORS) est une fonctionnalité de navigateur qui autorise le domaine cible à indiquer au navigateur que vous approuvez les requêtes en provenance du domaine source.

Supposons, par exemple, qu’une application web exécutée dans Azure envoie une requête pour une ressource à un compte de stockage Azure. L’application web est le domaine source, et le compte de stockage est le domaine cible. Vous pouvez configurer le partage des ressources cross-origin pour l’un des services du stockage Azure afin d’indiquer au navigateur web que les requêtes provenant du domaine source sont approuvées par le stockage Azure. Pour plus d’informations sur le partage des ressources cross-origin, consultez [Prise en charge du partage des ressources cross-origin (CORS) pour le stockage Azure](/rest/api/storageservices/Cross-Origin-Resource-Sharing--CORS--Support-for-the-Azure-Storage-Services).  
  
Les signatures d’accès partagé et le partage des ressources cross-origin vous aident à éviter une charge inutile sur votre application web.  

## <a name="batch-transactions"></a>Transactions par lots

Le service de Table prend en charge les transactions par lots sur les entités qui se trouvent dans la même table et qui appartiennent au même groupe de partitions. Pour plus d’informations, consultez [Effectuer des transactions de groupe d’entités](/rest/api/storageservices/performing-entity-group-transactions).

## <a name="net-configuration"></a>Configuration .NET

Si vous utilisez .NET Framework, vous trouverez dans cette section plusieurs paramètres de configuration rapide que vous pouvez appliquer pour améliorer sensiblement les performances.  Si vous utilisez un autre langage, vérifiez si des concepts similaires y sont associés.  

### <a name="use-net-core"></a>Utiliser .NET Core

Développez vos applications du stockage Azure avec .NET Core 2.1 ou ultérieur pour tirer parti des performances améliorées. L’utilisation de .NET Core 3.x est recommandée dans la mesure du possible.

Pour plus d’informations sur les améliorations des performances dans .NET Core, consultez les billets de blog suivants :

- [Performance Improvements in .NET Core 3.0](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-3-0/)
- [Performance Improvements in .NET Core 2.1](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-2-1/)

### <a name="increase-default-connection-limit"></a>Augmenter la limite de connexions par défaut

Dans .NET, le code suivant augmente la limite de connexions par défaut (qui est généralement de 2 dans un environnement client ou de 10 dans un environnement de serveur) à 100. En règle générale, vous devez définir la valeur sur environ le nombre de threads utilisés par votre application.  

```csharp
ServicePointManager.DefaultConnectionLimit = 100; //(Or More)  
```

Définissez la limite de connexions avant d’ouvrir une connexion.  

Pour les autres langages de programmation, voir la documentation correspondante pour savoir comment définir la limite de connexions.  

Pour plus d’informations, consultez le billet de blog [Services web : connexions simultanées](/archive/blogs/darrenj/web-services-concurrent-connections).  

### <a name="increase-minimum-number-of-threads"></a>Augmenter le nombre minimal de threads

Si vous utilisez des appels synchrones avec des tâches asynchrones, vous aurez peut-être besoin d’augmenter le nombre de threads dans le pool de threads :

```csharp
ThreadPool.SetMinThreads(100,100); //(Determine the right number for your application)  
```

Pour plus d’informations, consultez la méthode [ThreadPool.SetMinThreads](/dotnet/api/system.threading.threadpool.setminthreads).  

## <a name="unbounded-parallelism"></a>Parallélisme illimité

Même si le parallélisme peut être très utile pour les performances, soyez prudent lorsque vous utilisez le parallélisme illimité, car il n’existe aucune limite concernant le nombre de threads ou de requêtes parallèles. Veillez à limiter les requêtes parallèles pour charger ou télécharger des données, pour accéder à plusieurs partitions dans un même compte de stockage ou pour accéder à plusieurs éléments d’une même partition. Si vous optez pour un parallélisme illimité, votre application peut dépasser les capacités de l’appareil client ou les objectifs de scalabilité du compte de stockage, ce qui se traduit par des temps de latence plus importants et par une limitation.  

## <a name="client-libraries-and-tools"></a>Outils et bibliothèques clientes

Pour des performances optimales, utilisez toujours les bibliothèques clientes et les outils fournis par Microsoft les plus récents. Les bibliothèques clientes du stockage Azure sont disponibles pour plusieurs langages. Le stockage Azure prend également en charge PowerShell et Azure CLI. Microsoft s’attelle au développement de ces outils et de ces bibliothèques clientes dans une optique de performances. Il veille à leur mise à jour continue avec les versions de service les plus récentes et s’assure qu’ils répondent, en interne, à la plupart des pratiques validées concernant les performances.

## <a name="handle-service-errors"></a>Gérer les erreurs de service

Le stockage Azure retourne une erreur lorsque le service ne peut pas traiter une requête. Le fait de comprendre les erreurs qui peuvent être retournées par le stockage Azure dans un scénario donné permet d’optimiser les performances.

### <a name="timeout-and-server-busy-errors"></a>Erreurs d’expiration de délai et de serveur occupé

Le stockage Azure peut limiter votre application si celle-ci s’approche des limites de scalabilité. Dans certains cas, le stockage Azure peut ne pas être en mesure de traiter une requête en raison d’un problème temporaire. Dans les deux cas, le service peut retourner une erreur 503 (Serveur occupé) ou une erreur 500 (Délai expiré). Ces erreurs peuvent également se produire si le service rééquilibre les partitions de données pour permettre un débit plus élevé. L’application cliente doit généralement retenter l’opération qui provoque l’une de ces erreurs. Cependant, si le stockage Azure limite votre application en raison d’un dépassement des objectifs de scalabilité, ou si le service n’a pas été en mesure de répondre à la requête pour une autre raison, le fait d’effectuer des nouvelles tentatives agressives ne fait généralement qu’aggraver le problème. Il est recommandé d’utiliser une stratégie de nouvelle tentative de type backoff exponentiel. Les bibliothèques clientes adopteront par défaut ce comportement. Votre application peut, par exemple, effectuer une nouvelle tentative après 2 secondes, puis 4 secondes, 10 secondes, 30 secondes avant d’abandonner complètement. De cette façon, votre application réduit considérablement la charge sur le service, au lieu d’aggraver un comportement susceptible d’entraîner une limitation.  

Les erreurs de connectivité ne sont pas dues à une limitation et sont généralement temporaires. Dès lors, de nouvelles tentatives peuvent être effectuées immédiatement.  

### <a name="non-retryable-errors"></a>Erreurs non renouvelables

Les bibliothèques clientes gèrent les nouvelles tentatives en sachant quelles erreurs peuvent être retentées. Toutefois, si vous appelez directement l’API REST du stockage Azure, il y a des erreurs que vous ne devez pas renouveler. Par exemple, une erreur 400 (Requête incorrecte) indique que l’application cliente a envoyé une requête qui n’a pas pu être traitée, car elle n’était pas au format attendu. Le fait de renvoyer cette requête générera à chaque fois la même réponse, il ne sert donc à rien de la renvoyer. Si vous appelez directement l’API REST du stockage Azure, tenez compte des erreurs potentielles et déterminez si elles doivent être retentées.

Pour plus d’informations sur les codes d’erreur du stockage Azure, consultez [Codes d’état et d’erreur](/rest/api/storageservices/status-and-error-codes2).

## <a name="configuration"></a>Configuration

Cette section décrit les paramètres de configuration rapide que vous pouvez utiliser pour améliorer sensiblement les performances du service de Table :

### <a name="use-json"></a>Avec JSON

Depuis la version 2013-08-15 du service de stockage, le service de Table prend en charge l’utilisation de JSON plutôt que le format AtomPub XML pour transférer des données de table. L’utilisation du format JSON permet de réduire la taille de la charge utile de quelque 75 % et d’améliorer sensiblement les performances de votre application.

Pour plus d’informations, consultez [Tables Microsoft Azure : présentation de JSON](/archive/blogs/windowsazurestorage/windows-azure-tables-introducing-json) et [Format de charge utile pour les opérations du service de Table](/rest/api/storageservices/Payload-Format-for-Table-Service-Operations).

### <a name="disable-nagle"></a>Désactiver Nagle

L’algorithme de Nagle est utilisé à grande échelle sur les réseaux TCP/IP en vue d’améliorer les performances du réseau. Cependant, il n’est pas idéal dans toutes les situations (c’est le cas, par exemple, dans les environnements très interactifs). L’algorithme Nagle a un impact négatif sur les performances des requêtes adressées aux services Table et File d’attente Azure. Vous devez donc le désactiver si cela s’avère possible.

## <a name="schema"></a>schéma

Le mode de représentation et d’interrogation de vos données constitue le principal facteur ayant une incidence sur les performances du service de Table. Bien que chaque application soit différente, cette section énumère quelques pratiques générales concernant les points suivants :

- Conception de tables
- Requêtes efficaces
- Mises à jour de données efficaces

### <a name="tables-and-partitions"></a>Tables et partitions

Les tables sont divisées en partitions. Toutes les entités stockées dans une partition partagent la même clé de partition et sont associées à une clé de ligne pour les identifier dans cette partition. Les partitions offrent des avantages, mais elles s’accompagnent également de limites d’extensibilité.

- Avantages : vous pouvez mettre à jour des entités d’une même partition au cours d’une seule transaction atomique par lots pouvant contenir jusqu’à 100 opérations de stockage distinctes (taille totale limite de 4 Mo). En partant du principe que le même nombre d’entités doit être récupéré, vous pouvez également interroger plus efficacement les données d’une seule partition que celles qui couvrent plusieurs partitions (vous trouverez d’autres conseils sur l’interrogation des données de table dans la suite de ce document).
- Limite d’extensibilité : l’accès aux entités stockées dans une seule partition ne peut pas faire l’objet d’un équilibrage de la charge, car les partitions prennent en charge les transactions atomiques par lots. C’est pourquoi l’objectif de scalabilité d’une partition de table individuelle est inférieur à celui du service de Table dans son ensemble.

Compte tenu des caractéristiques des tables et des partitions, il est conseillé d’adopter les principes de conception suivants :

- Localisez les données que votre application cliente met à jour ou interroge fréquemment dans une même unité de travail logique au sein d’une même partition. Par exemple, localisez les données d’une même partition si votre application regroupe des écritures ou si vous effectuez des opérations de traitement par lots atomiques. Ajoutons encore que les données d’une seule partition peuvent être interrogées plus efficacement dans une seule requête que les données de plusieurs partitions.
- Localisez les données que votre application cliente n’insère pas, ne met pas à jour ou n’interroge pas dans une même unité de travail logique (requête unique ou mise à jour par lots) sur des partitions distinctes. N’oubliez pas que le nombre de clés de partition d’une même table n’est pas limité. Le fait qu’il y ait plusieurs millions de clés de partition ne constitue donc pas un problème et n’a aucune incidence sur les performances. Par exemple, si votre application est un site web populaire auquel les utilisateurs doivent se connecter, il peut être judicieux de choisir l’ID utilisateur comme clé de partition.

#### <a name="hot-partitions"></a>Partitions actives

On appelle « partition active » une partition qui reçoit un pourcentage disproportionné du trafic vers un compte et qui ne peut pas faire l’objet d’un équilibrage de la charge, car il s’agit d’une partition unique. En règle générale, la création des partitions actives s’effectue de l’une des façons suivantes :

#### <a name="append-only-and-prepend-only-patterns"></a>Modèles « Ajouter après uniquement » et « Ajouter avant uniquement »

Avec le modèle « Ajouter après uniquement », l’intégralité (ou la grande majorité) du trafic à destination d’une clé de partition donnée augmente et diminue en fonction de l’heure. Par exemple, supposons que votre application utilise la date du jour comme clé de partition pour les données de journal. De cette façons, toutes les insertions sont placées dans la dernière partition de votre table et le système ne peut pas équilibrer la charge correctement. Si le trafic à destination de cette partition dépasse l’objectif d’extensibilité au niveau de la partition, cela se traduit par une limitation. Il est préférable de s’assurer que le trafic est envoyé vers plusieurs partitions, afin de permettre l’équilibrage de la charge des demandes sur votre table.

#### <a name="high-traffic-data"></a>Données à trafic élevé

Si votre schéma de partitionnement donne lieu à une seule partition qui comporte simplement les données qui sont beaucoup plus utilisées que d’autres partitions, le phénomène de limitation risque également de se présenter à mesure que cette partition unique s’approche de son objectif d’extensibilité. Il est conseillé de faire en sorte que votre schéma de partitionnement ne génère pas une partition unique qui s’approche des objectifs d’extensibilité.

### <a name="querying"></a>Interrogation

Cette section décrit les pratiques validées concernant l’interrogation du service de Table.

#### <a name="query-scope"></a>Étendue de requête

Pour spécifier la plage des entités à interroger, vous pouvez procéder de plusieurs façons. La liste ci-dessous décrit chaque option pour l’étendue de requête.

- **Requêtes de point :** une requête de point récupère exactement une entité en spécifiant à la fois la clé de partition et la clé de ligne de l’entité à récupérer. Ces requêtes sont efficaces et nous vous conseillons de les utiliser dans la mesure du possible.
- **Requêtes de partition :** Une requête de partition récupère un jeu de données qui partagent une clé de partition commune. En règle générale, cette requête spécifie une plage de valeurs de clé de ligne ou une plage de valeurs pour une propriété d’entité, en plus d’une clé de partition. ces requêtes sont moins efficaces que les requêtes de point et doivent être utilisées avec modération.
- **Requêtes de table :** Une requête de table récupère un jeu d’entités ne partageant pas une clé de partition commune. Les requêtes de ce type ne sont pas efficaces et il est conseillé de les éviter dans la mesure du possible.

En règle générale, il est conseillé d’éviter les analyses (requêtes d’une taille supérieure à une seule entité). Cependant, si vous devez procéder de la sorte, tâchez d’organiser vos données de façon à ce que les analyses les récupèrent sans qu’il faille examiner ou renvoyer de grandes quantités d’entités dont vous n’avez pas besoin.

#### <a name="query-density"></a>Densité des requêtes

S’agissant de l’efficacité des requêtes, un autre facteur important est le nombre d’entités renvoyées par rapport au nombre d’entités analysées pour obtenir le jeu renvoyé. Si votre application effectue une requête de table avec un filtre pour une valeur de propriété partagée par seulement 1 % des données, la requête analyse 100 entités pour chaque entité renvoyée. Les objectifs de scalabilité de table abordés précédemment sont liés au nombre d’entités analysées et non au nombre d’entités retournées : une densité de requête faible peut facilement générer un service de Table qui limite votre application, car il convient d’analyser de nombreuses entités pour récupérer l’entité que vous recherchez. Pour savoir comment éviter la limitation, consultez la section intitulée [Dénormalisation ](#denormalization).

#### <a name="limiting-the-amount-of-data-returned"></a>Limitation du volume de données renvoyé

Lorsque vous savez qu’une requête va retourner des entités dont vous n’avez pas besoin dans l’application cliente, pensez à utiliser un filtre afin de réduire la taille du jeu retourné. Bien que les entités non renvoyées au client soient comptabilisées dans les limites d’extensibilité, les performances de votre application s’en trouveront améliorées, compte tenu de la taille réduite de charge utile du réseau et de la réduction du nombre d’entités traitées par votre application cliente. N’oubliez pas que les objectifs de scalabilité font référence au nombre d’entités analysées, et donc, qu’il se peut qu’une requête qui exclut de nombreuses entités donne toujours lieu à une limitation, même si le nombre d’entités retournées est faible. Pour plus d’informations sur l’efficacité des requêtes, consultez la section intitulée [Densité des requêtes](#query-density).

Si votre application cliente nécessite seulement un jeu limité de propriétés des entités de votre table, vous pouvez utiliser la projection pour limiter la taille du jeu de données renvoyé. Comme c’est le cas avec le filtrage, la projection permet de réduire la charge réseau et le traitement du client.

#### <a name="denormalization"></a>Dénormalisation

Contrairement à l’utilisation des bases de données relationnelles, les pratiques éprouvées pour interroger efficacement des données de table entraînent leur dénormalisation. Cette opération consiste à dupliquer les mêmes données dans plusieurs entités (une pour chaque clé utilisable pour rechercher les données) afin de réduire le nombre d’entités qu’une requête doit analyser pour rechercher les données dont le client a besoin, plutôt que d’analyser de grandes quantités d’entités pour rechercher les données dont votre application a besoin. Sur un site web de commerce électronique, par exemple, vous pouvez rechercher une commande en fonction de l’ID du client (rechercher les commandes de ce client) et de la date (rechercher les commandes passées à une date donnée). Dans le stockage de tables, il est préférable de stocker l’entité (ou une référence à celle-ci) à deux reprises : une fois avec le nom de la table, la clé de partition et la clé de ligne afin de faciliter la recherche en fonction de l’ID de client, et une fois pour faciliter la recherche en fonction de la date.  

### <a name="insert-update-and-delete"></a>Insérer, mettre à jour et supprimer

Cette section décrit les pratiques validées concernant la modification des entités stockées dans le service de Table.  

#### <a name="batching"></a>Traitement par lot

Les transactions par lots sont appelées transactions de groupe d’entités dans le stockage Azure. Toutes les opérations d’une transaction de groupe d’entités doivent se trouver sur une même partition d’une même table. Lorsque cela s’avère possible, utilisez des transactions de groupe d’entités pour effectuer des opérations d’insertion, de mise à jour et de suppression par lots. L’utilisation de transactions de groupe d’entités réduit le nombre d’allers-retours entre votre application cliente et le serveur, ainsi que le nombre de transactions facturables (une transaction de groupe d’entités est comptabilisée comme une seule transaction à des fins de facturation et peut contenir 100 opérations de stockage), et autorise les mises à jour atomiques (dans une transaction de groupe d’entités, toutes les opérations réussissent ou échouent). L’utilisation des transactions de groupe d’entités peut apporter des avantages non négligeables dans les environnements où les temps de latence sont élevés, tels que les appareils mobiles.  

#### <a name="upsert"></a>Upsert

Lorsque cela s'avère possible, il est conseillé d'utiliser des opérations de table **Upsert** . Il existe deux types d’opération **Upsert** ; tous deux peuvent se révéler plus efficaces que les opérations **Insert** et **Update** classiques :  

- **InsertOrMerge** : Utilisez cette opération lorsque vous souhaitez charger un sous-ensemble des propriétés de l’entité, mais ne savez pas si cette dernière existe déjà. Si elle existe, cet appel met à jour les propriétés incluses dans l’opération **Upsert** et laisse toutes les propriétés existantes en l’état. Si elle n’existe pas, cet appel insère la nouvelle entité. Cela revient à utiliser la projection dans une requête, en ce sens que vous devez simplement télécharger les propriétés qui sont modifiées.
- **InsertOrReplace** : Utilisez cette opération lorsque vous souhaitez charger une toute nouvelle entité, mais ne savez pas si cette dernière existe déjà. Utilisez cette opération lorsque vous n’avez aucun doute quant à la qualité de la nouvelle entité chargée, car celle-ci écrase complètement l’ancienne entité. Vous souhaitez, par exemple, mettre à jour l’entité qui stocke l’emplacement actuel d’un utilisateur et ce, que l’application ait déjà stocké ou non des données d’emplacement pour cet utilisateur ; la nouvelle entité d’emplacement est complète et vous n’avez besoin d’aucune information d’une entité précédente.

#### <a name="storing-data-series-in-a-single-entity"></a>Stockage de séries de données dans une même entité

Parfois, une application stocke une série de données fréquemment nécessaires pour tout récupérer à la fois : par exemple, une application peut suivre l’utilisation du processeur au fil du temps pour tracer un graphique propagée des données à partir des dernières 24 heures. Une méthode consiste à disposer d’une entité de table par heure, chaque entité représentant alors une heure donnée et stockant l’utilisation du processeur au cours de cette période. Pour représenter ces données sur un graphique, l’application doit récupérer les entités qui contiennent les données des 24 dernières heures.  

Sinon, votre application peut stocker l'utilisation du processeur pour chaque heure sous la forme d'une propriété distincte d'une entité unique : pour mettre à jour chaque heure, votre application peut utiliser un seul appel **InsertOrMerge Upsert** pour mettre à jour la valeur de la dernière heure. Pour représenter les données sur un graphique, l’application doit récupérer une seule entité au lieu de 24, générant ainsi une requête efficace. Pour plus d’informations sur l’efficacité des requêtes, consultez la section intitulée [Étendue des requêtes](#query-scope).

#### <a name="storing-structured-data-in-blobs"></a>Stockage de données structurées dans des objets blob

Si vous effectuez des insertions par lot, puis récupérez des plages d’entités, utilisez des objets blob plutôt que des tables. Un fichier journal constitue un parfait exemple. Vous pouvez regrouper plusieurs minutes de journalisation et les insérer, puis récupérer plusieurs minutes de journalisation à la fois. Dans ce cas, les performances sont meilleures si vous utilisez des objets blob plutôt que des tables, dans la mesure où vous pouvez réduire de manière significative le nombre d’objets écrits ou lus, ainsi que le nombre de requêtes qui doivent être effectuées.  

## <a name="next-steps"></a>Étapes suivantes

- [Objectifs d'extensibilité et de performances du service Stockage Table](scalability-targets.md)
- [Objectifs d’extensibilité et de performances pour les comptes de stockage standard](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2ftables%2ftoc.json)
- [Codes d’état et d’erreur](/rest/api/storageservices/Status-and-Error-Codes2)