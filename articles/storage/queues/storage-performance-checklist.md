---
title: Check-list des performances et de la scalabilité pour le Stockage File d’attente – Stockage Azure
description: Check-list des pratiques validées pour le Stockage File d’attente dans le développement d’applications performantes.
author: tamram
services: storage
ms.author: tamram
ms.date: 10/10/2019
ms.topic: overview
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-csharp
ms.openlocfilehash: 4040a81d5b509ddbdd355953e28721a7c9fccfb8
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585664"
---
<!-- docutune:casing "Timeout and Server Busy errors" -->

# <a name="performance-and-scalability-checklist-for-queue-storage"></a>Check-list des performances et de la scalabilité pour le Stockage File d’attente

Microsoft a mis au point un certain nombre de pratiques qui ont fait leurs preuves pour le développement d’applications hautes performances avec le service Stockage File d’attente. Cette check-list fournit des pratiques clés que les développeurs peuvent utiliser pour optimiser les performances. Gardez ces pratiques à l’esprit lorsque vous concevez votre application et tout au long du processus de développement.

Le stockage Azure a des objectifs de scalabilité et de performances pour la capacité, le taux de transactions et la bande passante. Pour plus d’informations sur les cibles de scalabilité de Stockage Azure, consultez [Cibles de scalabilité et de performances pour les comptes de stockage standard](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json) et [Cibles de scalabilité et de performances pour le Stockage File d’attente](scalability-targets.md).

## <a name="checklist"></a>Liste de contrôle

Cet article fournit la liste des pratiques validées concernant les performances. Vous pouvez suivre ces pratiques lors du développement de votre application Stockage File d’attente.

| Terminé | Category | Considérations relatives à la conception |
|--|--|--|
| &nbsp; | Objectifs de scalabilité | [Pouvez-vous concevoir votre application pour ne pas qu’elle utilise plus que le nombre maximal de comptes de stockage ?](#maximum-number-of-storage-accounts) |
| &nbsp; | Objectifs de scalabilité | [Cherchez-vous à ne pas vous approcher des limites de capacité et de transactions ?](#capacity-and-transaction-targets) |
| &nbsp; | Mise en réseau | [Les appareils côté client disposent-ils d’une bande passante suffisamment large et d’une latence suffisamment faible pour parvenir aux performances nécessaires ?](#throughput) |
| &nbsp; | Mise en réseau | [La qualité du lien réseau côté client est-elle suffisamment élevée ?](#link-quality) |
| &nbsp; | Mise en réseau | [L’application cliente se trouve-t-elle dans la même région que le compte de stockage ?](#location) |
| &nbsp; | Accès direct au client | [Utilisez-vous des signatures d’accès partagé (SAP) et un partage des ressources cross-origin (CORS) pour permettre l’accès direct au stockage Azure ?](#sas-and-cors) |
| &nbsp; | Configuration .NET | [Utilisez-vous .NET Core 2.1 ou ultérieur pour obtenir des performances optimales ?](#use-net-core) |
| &nbsp; | Configuration .NET | [Avez-vous configuré votre client pour qu'il utilise un nombre suffisant de connexions simultanées ?](#increase-default-connection-limit) |
| &nbsp; | Configuration .NET | [Pour les applications .NET, avez-vous configuré .NET pour qu’il utilise un nombre suffisant de threads ?](#increase-the-minimum-number-of-threads) |
| &nbsp; | Parallélisme | [Avez-vous vérifié que le parallélisme est limité de façon appropriée, de manière à ne pas surcharger les fonctionnalités de votre client ni s’approcher des objectifs de scalabilité ?](#unbounded-parallelism) |
| &nbsp; | Outils | [Utilisez-vous la version la plus récente des outils et bibliothèques clientes fournis par Microsoft ?](#client-libraries-and-tools) |
| &nbsp; | Nouvelle tentatives | [Utilisez-vous une stratégie de nouvelles tentatives avec backoff exponentiel pour les erreurs de limitation et les délais d’expiration ?](#timeout-and-server-busy-errors) |
| &nbsp; | Nouvelle tentatives | [Votre application empêche-t-elle les nouvelles tentatives pour les erreurs non renouvelables ?](#non-retryable-errors) |
| &nbsp; | Configuration | [Avez-vous désactivé l’algorithme de Nagle pour améliorer les performances des petites requêtes ?](#disable-nagles-algorithm) |
| &nbsp; | Taille des messages | [Vos messages sont-ils compacts pour améliorer les performances de la file d'attente ?](#message-size) |
| &nbsp; | Récupération en bloc | [Récupérez-vous plusieurs messages dans une même opération de récupération ?](#batch-retrieval) |
| &nbsp; | Fréquence d’interrogation | [Effectuez-vous des interrogations suffisamment fréquentes pour réduire la latence perçue de votre application ?](#queue-polling-interval) |
| &nbsp; | Message de mise à jour | [Effectuez-vous une opération de mise à jour de message pour stocker la progression lors du traitement des messages et éviter ainsi de devoir retraiter l’intégralité du message en cas d’erreur ?](#perform-an-update-message-operation) |
| &nbsp; | Architecture | [Utilisez-vous des files d'attente pour rendre toute votre application plus extensible en excluant les charges de travail de longue durée du chemin critique et pour les faire ensuite évoluer séparément ?](#application-architecture) |

## <a name="scalability-targets"></a>Objectifs de scalabilité

Si votre application s’approche de l’un des objectifs d’extensibilité, voire le dépasse, une limitation ou des latences de transaction accrues peuvent survenir. Lorsque le stockage Azure limite votre application, le service commence à retourner les codes d’erreur 503 (`Server Busy`) ou 500 (`Operation Timeout`). Pour améliorer les performances de votre application, il est important d’éviter de telles erreurs en restant dans les limites des objectifs de scalabilité.

Pour plus d’informations sur les objectifs de scalabilité du service Stockage File d’attente, consultez [Objectifs de scalabilité et de performances du Stockage Azure](./scalability-targets.md#scale-targets-for-queue-storage).

### <a name="maximum-number-of-storage-accounts"></a>Nombre maximal de comptes de stockage

Si vous atteignez le nombre maximal de comptes de stockage autorisés pour une combinaison abonnement-région, utilisez-vous plusieurs comptes de stockage afin d’effectuer un partitionnement en vue d’augmenter les entrées, les sorties, les opérations d’E/S par seconde (IOPS) ou la capacité ? Dans ce scénario, Microsoft recommande, si possible, de tirer parti de l’augmentation du nombre maximal de comptes de stockage, afin de réduire le nombre de comptes de stockage nécessaires pour votre charge de travail. Contactez le [support Azure](https://azure.microsoft.com/support/options/) pour demander l’augmentation des limites de votre compte de stockage. Pour plus d’informations, consultez [Annonce de comptes de stockage plus grands et à plus grande échelle](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/).

### <a name="capacity-and-transaction-targets"></a>Objectifs de capacité et de transaction

Si votre application s’approche des objectifs d’extensibilité d’un seul compte de stockage, pensez à appliquer l’une des méthodes suivantes :

- Si les objectifs de scalabilité des files d’attente sont insuffisants pour votre application, utilisez plusieurs files d’attente et répartissez les messages entre elles.
- Réexaminez la charge de travail à cause de laquelle votre application s’approche de l’objectif d’extensibilité ou le dépasse. Est-il possible de la concevoir différemment pour qu’elle utilise moins de bande passante ou de capacité, ou moins de transactions ?
- Si votre application doit dépasser l’un des objectifs de scalabilité, vous devez créer plusieurs comptes de stockage et partitionner vos données d’application sur ces comptes de stockage. Si vous optez pour ce modèle, veillez à concevoir votre application de telle sorte que vous puissiez ajouter davantage de comptes de stockage à l’avenir en vue de l’équilibrage de la charge. Aucun coût autre que l’utilisation (données stockées, transactions effectuées, données transférées, etc.) n’est associé à ces comptes de stockage.
- Si votre application s’approche des objectifs de bande passante, pensez à compresser les données côté client afin de réduire la bande passante nécessaire à l’envoi des données au stockage Azure. Même si la compression des données peut réduire la bande passante et améliorer les performances du réseau, elle présente des inconvénients. Évaluez l’impact sur les performances qu’entraînent les exigences de traitement supplémentaires liées à la compression et à la décompression des données côté client. Gardez à l’esprit que le stockage de données compressées peut compliquer la résolution des problèmes, car il peut être plus difficile de voir les données à l’aide d’outils standard.
- Si votre application s’approche des objectifs de scalabilité, vérifiez que vous utilisez bien un backoff exponentiel pour les nouvelles tentatives. Il est préférable d’éviter de s’approcher des objectifs de scalabilité en implémentant les recommandations fournies dans cet article. Toutefois, l’utilisation d’un backoff exponentiel pour les nouvelles tentatives va empêcher votre application d’effectuer une nouvelle tentative rapidement, ce qui pourrait compliquer la limitation. Pour plus d’informations, consultez la section [Erreurs d’expiration de délai et de serveur occupé](#timeout-and-server-busy-errors).

## <a name="networking"></a>Mise en réseau

Les contraintes de réseau physiques de l’application peuvent avoir un impact majeur sur les performances. Les sections suivantes abordent certaines des limitations auxquelles les utilisateurs peuvent être confrontés.

### <a name="client-network-capability"></a>Fonctionnalités réseau du client

La bande passante et la qualité du lien réseau jouent un rôle important dans les performances de l’application, comme l’expliquent les sections suivantes.

#### <a name="throughput"></a>Débit

Dans le cas de la bande passante, le problème est souvent dû aux capacités du client. Dans le cas des instances Azure plus importantes, les cartes réseau présentent une capacité supérieure. Si vous avez besoin de limites réseau plus élevées à partir d’un seul ordinateur, vous devez donc envisager d’utiliser une instance plus grande ou davantage de machines virtuelles. Si vous accédez au stockage Azure à partir d’une application locale, alors la même règle s’applique : comprendre les capacités réseau de l’appareil client et la connectivité réseau à l’emplacement de stockage Azure, et l’améliorer selon vos besoins ou concevoir votre application conformément à ces capacités.

#### <a name="link-quality"></a>Qualité du lien

Comme c’est le cas pour toute utilisation du réseau, n’oubliez pas que les conditions réseau qui génèrent des erreurs et une perte de paquets ralentissent le débit effectif. L’utilisation de Wireshark ou de Moniteur réseau peut vous aider à diagnostiquer ce problème.

### <a name="location"></a>Emplacement

Dans un environnement distribué, le fait de placer le client à proximité du serveur se traduit par des performances optimales. Pour accéder à Azure Storage avec un minimum de latence, votre client doit idéalement se trouver dans la même région Azure. Par exemple, si vous disposez d’une application web Azure qui utilise le stockage Azure, tous deux doivent se trouver dans la même région (USA Ouest ou Asie Sud-Est, par exemple). Cela réduit à la fois la latence et les coûts, puisque l’utilisation de la bande passante dans une seule région était gratuite.

Si des applications clientes accèdent au stockage Azure sans être hébergées dans Azure (c’est le cas, par exemple, des applications pour appareil mobile ou des services d’entreprise locaux), le fait de placer le compte de stockage dans une région proche de ces appareils peut réduire la latence. Si vos clients sont distribués à grande échelle (par exemple, certains en Amérique du Nord et d’autres en Europe), il est conseillé d’utiliser un compte de stockage pour chaque région. Cette méthode est plus facile à implémenter si les données stockées par l’application sont propres à certains utilisateurs, et elle ne nécessite pas de réplication des données entre différents comptes de stockage.

## <a name="sas-and-cors"></a>SAP et CORS

Supposons que vous deviez autoriser du code, tel que du code JavaScript qui s’exécute dans le navigateur web d’un utilisateur ou dans une application de téléphone mobile, à accéder aux données du stockage Azure. L’une des méthodes possibles consiste à créer une application de service qui serve de proxy. L’appareil de l’utilisateur s’authentifie auprès du service, qui à son tour autorise l’accès aux ressources du stockage Azure. Vous évitez ainsi d’exposer vos clés de compte de stockage sur des appareils non sécurisés. Cependant, cette méthode entraîne une surcharge importante pour l’application du service, dans la mesure où toutes les données transférées entre l’appareil de l’utilisateur et le stockage Azure doivent transiter par l’application du service.

Vous pouvez éviter d’utiliser une application de service en tant que proxy pour le stockage Azure en utilisant des signatures d’accès partagé (SAS). Avec les signatures d’accès partagé, vous pouvez autoriser l’appareil de votre utilisateur à adresser directement des requêtes au stockage Azure par le biais d’un jeton à accès limité. Par exemple, si un utilisateur souhaite charger une photo vers votre application, votre application de service peut générer une signature d’accès partagé et l’envoyer à l’appareil de l’utilisateur. Le jeton SAS peut accorder l’autorisation d’écrire dans une ressource du stockage Azure pendant un intervalle de temps spécifié, au bout duquel le jeton SAS expire. Pour plus d’informations sur les SAS, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAS)](../common/storage-sas-overview.md).

En règle générale, un navigateur web n’autorise pas le code JavaScript d’une page hébergée par un site web d’un domaine à effectuer certaines opérations, telles que les opérations d’écriture, sur un autre domaine. Connue sous le nom de stratégie de même origine, cette stratégie empêche un script malveillant d’une page d’obtenir l’accès aux données d’une autre page web. Toutefois, la stratégie de même origine peut représenter un obstacle lorsque vous créez une solution dans le cloud. Le partage des ressources cross-origin (CORS) est une fonctionnalité de navigateur qui autorise le domaine cible à indiquer au navigateur que vous approuvez les requêtes en provenance du domaine source.

Supposons, par exemple, qu’une application web exécutée dans Azure envoie une requête pour une ressource à un compte de stockage Azure. L’application web est le domaine source, et le compte de stockage est le domaine cible. Vous pouvez configurer le partage des ressources cross-origin pour l’un des services du stockage Azure afin d’indiquer au navigateur web que les requêtes provenant du domaine source sont approuvées par le stockage Azure. Pour plus d’informations sur le partage des ressources cross-origin, consultez [Prise en charge du partage des ressources cross-origin (CORS) pour le stockage Azure](/rest/api/storageservices/Cross-Origin-Resource-Sharing--CORS--Support-for-the-Azure-Storage-Services).

Les signatures d’accès partagé et le partage des ressources cross-origin vous aident à éviter une charge inutile sur votre application web.

## <a name="net-configuration"></a>Configuration .NET

Si vous utilisez .NET Framework, vous trouverez dans cette section plusieurs paramètres de configuration rapide que vous pouvez appliquer pour améliorer sensiblement les performances. Si vous utilisez un autre langage, vérifiez si des concepts similaires y sont associés.

### <a name="use-net-core"></a>Utiliser .NET Core

Développez vos applications du stockage Azure avec .NET Core 2.1 ou ultérieur pour tirer parti des performances améliorées. L’utilisation de .NET Core 3.x est recommandée dans la mesure du possible.

Pour plus d’informations sur les améliorations des performances dans .NET Core, consultez les billets de blog suivants :

- [Amélioration des performances dans .NET Core 3.0](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-3-0/)
- [Amélioration des performances dans .NET Core 2.1](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-2-1/)

### <a name="increase-default-connection-limit"></a>Augmenter la limite de connexions par défaut

Dans .NET, le code suivant augmente la limite de connexions par défaut (qui est généralement de 2 dans un environnement client ou de 10 dans un environnement de serveur) à 100. En règle générale, vous devez définir la valeur sur environ le nombre de threads utilisés par votre application.

```csharp
ServicePointManager.DefaultConnectionLimit = 100; //(Or More)  
```

Définissez la limite de connexions avant d’ouvrir une connexion.

Pour les autres langages de programmation, voir la documentation correspondante pour savoir comment définir la limite de connexions.

Pour plus d’informations, consultez le billet de blog [Services web : Connexions simultanées](/archive/blogs/darrenj/web-services-concurrent-connections).

### <a name="increase-the-minimum-number-of-threads"></a>Augmenter le nombre minimal de threads

Si vous utilisez des appels synchrones avec des tâches asynchrones, vous aurez peut-être besoin d’augmenter le nombre de threads dans le pool de threads :

```csharp
ThreadPool.SetMinThreads(100,100); //(Determine the right number for your application)  
```

Pour plus d’informations, reportez-vous à la méthode [`ThreadPool.SetMinThreads`](/dotnet/api/system.threading.threadpool.setminthreads).

## <a name="unbounded-parallelism"></a>Parallélisme illimité

Même si le parallélisme peut être très utile pour les performances, soyez prudent lorsque vous utilisez le parallélisme illimité, car il n’existe aucune limite concernant le nombre de threads ou de requêtes parallèles. Veillez à limiter les requêtes parallèles pour charger ou télécharger des données, pour accéder à plusieurs partitions dans un même compte de stockage ou pour accéder à plusieurs éléments d’une même partition. Si vous optez pour un parallélisme illimité, votre application peut dépasser les capacités de l’appareil client ou les objectifs de scalabilité du compte de stockage, ce qui se traduit par des temps de latence plus importants et par une limitation.

## <a name="client-libraries-and-tools"></a>Outils et bibliothèques clientes

Pour des performances optimales, utilisez toujours les bibliothèques clientes et les outils fournis par Microsoft les plus récents. Les bibliothèques clientes du stockage Azure sont disponibles pour plusieurs langages. Le stockage Azure prend également en charge PowerShell et Azure CLI. Microsoft s’attelle au développement de ces outils et de ces bibliothèques clientes dans une optique de performances. Il veille à leur mise à jour continue avec les versions de service les plus récentes et s’assure qu’ils répondent, en interne, à la plupart des pratiques validées concernant les performances. Pour plus d’informations, consultez la [documentation de référence du stockage Azure](./reference.md).

## <a name="handle-service-errors"></a>Gérer les erreurs de service

Le stockage Azure retourne une erreur lorsque le service ne peut pas traiter une requête. Le fait de comprendre les erreurs qui peuvent être retournées par le stockage Azure dans un scénario donné permet d’optimiser les performances.

### <a name="timeout-and-server-busy-errors"></a>Erreurs d’expiration de délai et de serveur occupé

Le stockage Azure peut limiter votre application si celle-ci s’approche des limites de scalabilité. Dans certains cas, le stockage Azure peut ne pas être en mesure de traiter une requête en raison d’un problème temporaire. Dans les deux cas, le service peut retourner une erreur 503 (`Server Busy`) ou une erreur 500 (`Timeout`). Ces erreurs peuvent également se produire si le service rééquilibre les partitions de données pour permettre un débit plus élevé. L’application cliente doit généralement retenter l’opération qui provoque l’une de ces erreurs. Cependant, si le stockage Azure limite votre application en raison d’un dépassement des objectifs de scalabilité, ou si le service n’a pas été en mesure de répondre à la requête pour une autre raison, le fait d’effectuer des nouvelles tentatives agressives ne fait généralement qu’aggraver le problème. Il est recommandé d’utiliser une stratégie de nouvelle tentative de type backoff exponentiel. Les bibliothèques clientes adopteront par défaut ce comportement. Votre application peut, par exemple, effectuer une nouvelle tentative après 2 secondes, puis 4 secondes, 10 secondes, 30 secondes avant d’abandonner complètement. De cette façon, votre application réduit considérablement la charge sur le service, au lieu d’aggraver un comportement susceptible d’entraîner une limitation.

Les erreurs de connectivité ne sont pas dues à une limitation et sont généralement temporaires. Dès lors, de nouvelles tentatives peuvent être effectuées immédiatement.

### <a name="non-retryable-errors"></a>Erreurs non renouvelables

Les bibliothèques clientes gèrent les nouvelles tentatives en sachant quelles erreurs peuvent être retentées. Toutefois, si vous appelez directement l’API REST du stockage Azure, il y a des erreurs que vous ne devez pas renouveler. Par exemple, une erreur 400 (`Bad Request`) indique que l’application cliente a envoyé une requête qui n’a pas pu être traitée, car elle n’était pas au format attendu. Le fait de renvoyer cette requête générera à chaque fois la même réponse, il ne sert donc à rien de la renvoyer. Si vous appelez directement l’API REST du stockage Azure, tenez compte des erreurs potentielles et déterminez si elles doivent être retentées.

Pour plus d’informations sur les codes d’erreur du stockage Azure, consultez [Codes d’état et d’erreur](/rest/api/storageservices/status-and-error-codes2).

## <a name="disable-nagles-algorithm"></a>Désactiver l’algorithme de Nagle

L’algorithme de Nagle est utilisé à grande échelle sur les réseaux TCP/IP en vue d’améliorer les performances du réseau. Cependant, il n’est pas idéal dans toutes les situations (c’est le cas, par exemple, dans les environnements très interactifs). L’algorithme de Nagle a un impact négatif sur les performances des requêtes adressées au Stockage Table Azure. Vous devez donc le désactiver si cela s’avère possible.

## <a name="message-size"></a>Taille des messages

Les performances et l’extensibilité des files d’attente diminuent quand la taille de message augmente. Dans un message, incluez uniquement les informations dont le destinataire a besoin.

## <a name="batch-retrieval"></a>La récupération par lots

Vous pouvez récupérer jusqu’à 32 messages d’une file d’attente en une seule opération. La récupération par lots contribue à réduire le nombre d’allers-retours avec l’application cliente, ce qui s’avère particulièrement utile pour les environnements où les temps de latence sont élevés, tels que les appareils mobiles.

## <a name="queue-polling-interval"></a>Intervalle d’interrogation de file d’attente

La plupart des applications interrogent une file d’attente pour les messages, ce qui peut représenter l’une des plus grandes sources de transactions pour cette application. Sélectionnez votre intervalle d’interrogation avec soin : une interrogation trop fréquente pourrait entraîner un rapprochement des objectifs d’extensibilité pour la file d’attente. Toutefois, à 200 000 transactions à 0,01 $ (au moment de la rédaction), un seul processeur interrogeant une fois par seconde pendant un mois reviendrait à moins de 15 cents. Le prix n’est donc généralement pas un facteur qui affecte le choix de l’intervalle d’interrogation.

Pour des informations tarifaires à jour, consultez [Tarifs du stockage Azure](https://azure.microsoft.com/pricing/details/storage/).

## <a name="perform-an-update-message-operation"></a>Effectuer une opération de mise à jour de message

Vous pouvez effectuer une opération de mise à jour de message pour augmenter le délai d’expiration de l’invisibilité ou pour mettre à jour les informations d’état d’un message. Cette méthode peut être plus efficace qu’un workflow qui transmet un travail d’une file d’attente à une autre, à chaque fin d’étape effectuée du travail. Votre application peut enregistrer l’état de la tâche dans le message, puis poursuivre le traitement, au lieu de replacer à chaque fois le message en file d’attente pour l’étape suivante. N’oubliez pas que chaque opération de mise à jour de message est comptabilisée dans le cadre de l’objectif de scalabilité.

## <a name="application-architecture"></a>Architecture de l’application

Utilisez des files d’attente pour rendre scalable l’architecture de votre application. Vous trouverez, dans les listes suivantes, les méthodes applicables pour accroître l’extensibilité de votre application :

- Vous pouvez utiliser des files d’attente pour créer des journaux de travaux en souffrance à traiter et éliminer des charges de travail de votre application. Vous pouvez, par exemple, placer en file d’attente des demandes d’utilisateurs portant sur l’exécution de tâches exigeant d’importantes ressources processeur, telles que le redimensionnement d’images téléchargées.
- Vous pouvez utiliser des files d’attente pour dissocier des parties de votre application, de manière à pouvoir les faire évoluer séparément. Un front-end web peut, par exemple, placer les résultats d’une enquête en file d’attente en vue de les stocker et de les analyser ultérieurement. Vous pouvez ajouter des instances du rôle de travail afin de traiter les données de file d’attente suivant les besoins.

## <a name="next-steps"></a>Étapes suivantes

- [Objectifs de scalabilité et de performances pour le Stockage File d’attente](scalability-targets.md)
- [Objectifs d’extensibilité et de performances pour les comptes de stockage standard](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)
- [Codes d’état et d’erreur](/rest/api/storageservices/Status-and-Error-Codes2)
