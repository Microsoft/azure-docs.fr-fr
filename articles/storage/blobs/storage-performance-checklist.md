---
title: Liste de contrôle des performances et de la scalabilité pour le stockage Blob - Stockage Azure
description: Liste de pratiques validées pour le stockage Blob en vue du développement d’applications performantes.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 10/10/2019
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 14da8b6cb695703f1881b6b0b9858772bde386c5
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544749"
---
# <a name="performance-and-scalability-checklist-for-blob-storage"></a>Liste de contrôle des performances et de la scalabilité pour le stockage Blob

Microsoft a mis au point un certain nombre de pratiques qui ont fait leurs preuves pour le développement d’applications hautes performances avec le stockage Blob. Cette check-list fournit des pratiques clés que les développeurs peuvent utiliser pour optimiser les performances. Gardez ces pratiques à l’esprit lorsque vous concevez votre application et tout au long du processus de développement.

Le stockage Azure a des objectifs de scalabilité et de performances pour la capacité, le taux de transactions et la bande passante. Pour plus d’informations sur les cibles d’extensibilité de Stockage Azure, consultez [Cibles d’extensibilité et de performances pour les comptes de stockage standard](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) et [Cibles d’extensibilité et de performances pour le stockage Blob](scalability-targets.md).

## <a name="checklist"></a>Liste de contrôle

Cet article fournit la liste des pratiques validées concernant les performances. Vous pouvez suivre ces pratiques lors du développement de votre application de stockage Blob.

| Terminé | Category | Considérations relatives à la conception |
| --- | --- | --- |
| &nbsp; |Objectifs de scalabilité |[Pouvez-vous concevoir votre application pour ne pas qu’elle utilise plus que le nombre maximal de comptes de stockage ?](#maximum-number-of-storage-accounts) |
| &nbsp; |Objectifs de scalabilité |[Cherchez-vous à ne pas vous approcher des limites de capacité et de transactions ?](#capacity-and-transaction-targets) |
| &nbsp; |Objectifs de scalabilité |[Existe-t-il un grand nombre de clients accédant simultanément à un objet blob unique ?](#multiple-clients-accessing-a-single-blob-concurrently) |
| &nbsp; |Objectifs de scalabilité |[Votre application respecte-t-elle les objectifs d’évolutivité pour un objet blob unique ?](#bandwidth-and-operations-per-blob) |
| &nbsp; |Partitionnement |[Votre convention d'affectation de noms est-elle conçue pour améliorer l'équilibrage de la charge ?](#partitioning) |
| &nbsp; |Mise en réseau |[Les appareils côté client disposent-ils d’une bande passante suffisamment large et d’une latence suffisamment faible pour parvenir aux performances nécessaires ?](#throughput) |
| &nbsp; |Mise en réseau |[La qualité du lien réseau côté client est-elle suffisamment élevée ?](#link-quality) |
| &nbsp; |Mise en réseau |[L’application cliente se trouve-t-elle dans la même région que le compte de stockage ?](#location) |
| &nbsp; |Accès direct au client |[Utilisez-vous des signatures d’accès partagé (SAP) et un partage des ressources cross-origin (CORS) pour permettre l’accès direct au stockage Azure ?](#sas-and-cors) |
| &nbsp; |Mise en cache |[La mise en cache des données de votre application est-elle souvent sollicitée et rarement modifiée ?](#reading-data) |
| &nbsp; |Mise en cache |[Votre application traite-t-elle les mises à jour par lots en effectuant la mise en cache sur le client, suivie du chargement dans des ensembles plus grands ?](#uploading-data-in-batches) |
| &nbsp; |Configuration .NET |[Utilisez-vous .NET Core 2.1 ou ultérieur pour obtenir des performances optimales ?](#use-net-core) |
| &nbsp; |Configuration .NET |[Avez-vous configuré votre client pour qu'il utilise un nombre suffisant de connexions simultanées ?](#increase-default-connection-limit) |
| &nbsp; |Configuration .NET |[Pour les applications .NET, avez-vous configuré .NET pour qu’il utilise un nombre suffisant de threads ?](#increase-minimum-number-of-threads) |
| &nbsp; |Parallélisme |[Avez-vous vérifié que le parallélisme est limité de façon appropriée, de manière à ne pas surcharger les fonctionnalités de votre client ni s’approcher des objectifs de scalabilité ?](#unbounded-parallelism) |
| &nbsp; |Outils |[Utilisez-vous la version la plus récente des outils et bibliothèques clientes fournis par Microsoft ?](#client-libraries-and-tools) |
| &nbsp; |Nouvelle tentatives |[Utilisez-vous une stratégie de nouvelles tentatives avec backoff exponentiel pour les erreurs de limitation et les délais d’expiration ?](#timeout-and-server-busy-errors) |
| &nbsp; |Nouvelle tentatives |[Votre application empêche-t-elle les nouvelles tentatives pour les erreurs non renouvelables ?](#non-retryable-errors) |
| &nbsp; |Copie d’objets blob |[La copie des objets blob s'effectue-t-elle de manière optimale ?](#blob-copy-apis) |
| &nbsp; |Copie d’objets blob |[Utilisez-vous la dernière version de AzCopy pour les opérations de copie en bloc ?](#use-azcopy) |
| &nbsp; |Copie d’objets blob |[Utilisez-vous la famille Azure Data Box pour importer des volumes importants de données ?](#use-azure-data-box) |
| &nbsp; |Distribution de contenu |[Utilisez-vous un CDN pour la distribution de contenu ?](#content-distribution) |
| &nbsp; |Utiliser les métadonnées |[Stockez-vous des métadonnées fréquemment utilisées concernant des objets blob dans leurs métadonnées ?](#use-metadata) |
| &nbsp; |Chargement rapide |[Lorsque vous essayez de télécharger rapidement un seul objet blob, téléchargez-vous les blocs en parallèle ?](#upload-one-large-blob-quickly) |
| &nbsp; |Chargement rapide |[Lorsque vous essayez de télécharger rapidement de nombreux objets blob, les téléchargez-vous en parallèle ?](#upload-many-blobs-quickly) |
| &nbsp; |Type d’objet blob |[Utilisez-vous des objets blob de pages ou de blocs lorsque cela s'avère approprié ?](#choose-the-correct-type-of-blob) |

## <a name="scalability-targets"></a>Objectifs de scalabilité

Si votre application s’approche de l’un des objectifs d’extensibilité, voire le dépasse, une limitation ou des latences de transaction accrues peuvent survenir. Lorsque le stockage Azure limite votre application, le service commence à retourner les codes d’erreur 503 (Serveur occupé) ou 500 (Délai d’expiration de l’opération). Pour améliorer les performances de votre application, il est important d’éviter de telles erreurs en restant dans les limites des objectifs de scalabilité.

Pour plus d’informations sur les objectifs de scalabilité concernant le service de File d’attente, consultez [Objectifs de scalabilité et de performances du Stockage Azure](../queues/scalability-targets.md#scale-targets-for-queue-storage).

### <a name="maximum-number-of-storage-accounts"></a>Nombre maximal de comptes de stockage

Si vous atteignez le nombre maximal de comptes de stockage autorisés pour une combinaison abonnement/région particulière, évaluez votre scénario et déterminez si l’une des conditions suivantes s’applique :

- Utilisez-vous des comptes de stockage pour stocker des disques non managés et ajouter ces disques à vos machines virtuelles ? Pour ce scénario, Microsoft recommande d’utiliser des disques managés. Les disques managés sont mis à l’échelle automatiquement et sans qu’il soit nécessaire de créer et de gérer des comptes de stockage individuels. Pour plus d’informations, consultez [Présentation des disques managés Azure](../../virtual-machines/managed-disks-overview.md)
- Utilisez-vous un compte de stockage par client, dans le but d’isoler les données ? Pour ce scénario, Microsoft recommande d’utiliser un conteneur d’objets blob pour chaque client plutôt qu’un compte de stockage entier. Stockage Azure vous permet désormais d’affecter des rôles Azure conteneur par conteneur. Pour plus d’informations, consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../common/storage-auth-aad-rbac-portal.md).
- Utilisez-vous plusieurs comptes de stockage de partition afin d’augmenter l’entrée, la sortie, les opérations d’E/S par seconde (IOPS) ou la capacité ? Dans ce scénario, Microsoft recommande, si possible, de tirer parti de l’augmentation du nombre maximal de comptes de stockage, afin de réduire le nombre de comptes de stockage nécessaires pour votre charge de travail. Contactez le [support Azure](https://azure.microsoft.com/support/options/) pour demander l’augmentation des limites de votre compte de stockage. Pour plus d’informations, consultez [Annonce de comptes de stockage plus grands et à plus grande échelle](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/).

### <a name="capacity-and-transaction-targets"></a>Objectifs de capacité et de transaction

Si votre application s’approche des objectifs d’extensibilité d’un seul compte de stockage, pensez à appliquer l’une des méthodes suivantes :  

- Si votre application atteint la cible de la transaction, envisagez d’utiliser des comptes de stockage d’objets blob de blocs, qui sont optimisés pour des taux de transactions élevés et une latence faible et cohérente. Pour plus d’informations, consultez [Vue d’ensemble des comptes de stockage Azure](../common/storage-account-overview.md).
- Réexaminez la charge de travail à cause de laquelle votre application s’approche de l’objectif d’extensibilité ou le dépasse. Est-il possible de la concevoir différemment pour qu’elle utilise moins de bande passante ou de capacité, ou moins de transactions ?
- Si votre application doit dépasser l’un des objectifs de scalabilité, vous devez créer plusieurs comptes de stockage et partitionner vos données d’application sur ces comptes de stockage. Si vous optez pour ce modèle, veillez à concevoir votre application de telle sorte que vous puissiez ajouter davantage de comptes de stockage à l’avenir en vue de l’équilibrage de la charge. Aucun coût autre que l’utilisation (données stockées, transactions effectuées, données transférées, etc.) n’est associé à ces comptes de stockage.
- Si votre application s’approche des objectifs de bande passante, pensez à compresser les données côté client afin de réduire la bande passante nécessaire à l’envoi des données au stockage Azure.
    Même si la compression des données peut réduire la bande passante et améliorer les performances du réseau, elle présente des inconvénients. Évaluez l’impact sur les performances qu’entraînent les exigences de traitement supplémentaires liées à la compression et à la décompression des données côté client. Gardez à l’esprit que le stockage de données compressées peut compliquer la résolution des problèmes, car il peut être plus difficile de voir les données à l’aide d’outils standard.
- Si votre application s’approche des objectifs de scalabilité, vérifiez que vous utilisez bien un backoff exponentiel pour les nouvelles tentatives. Il est préférable d’éviter de s’approcher des objectifs de scalabilité en implémentant les recommandations fournies dans cet article. Toutefois, l’utilisation d’un backoff exponentiel pour les nouvelles tentatives va empêcher votre application d’effectuer une nouvelle tentative rapidement, ce qui pourrait compliquer la limitation. Pour plus d’informations, consultez la section intitulée [Erreurs d’expiration de délai et de serveur occupé](#timeout-and-server-busy-errors).

### <a name="multiple-clients-accessing-a-single-blob-concurrently"></a>Plusieurs clients accédant simultanément à un seul objet blob

Si vous avez un grand nombre de clients qui accèdent simultanément à un seul objet blob, vous devrez prendre en compte les objectifs de scalabilité par objet blob et par compte de stockage. Le nombre exact de clients qui peuvent accéder à un objet blob unique varie en fonction de facteurs tels que le nombre de clients demandant l’accès à l’objet blob simultanément, la taille de l’objet blob et les conditions réseau.

Si l’objet blob peut être distribué par le biais d’un CDN, comme les images ou les vidéos provenant d’un site web, vous pouvez utiliser un CDN. Pour plus d’informations, consultez la section intitulée [Distribution de contenu](#content-distribution).

Dans d’autres scénarios tels que les simulations scientifiques où les données sont confidentielles, vous disposez de deux options. La première consiste à échelonner l’accès de votre charge de travail de façon à ce que l’objet blob soit accessible sur une période de temps au lieu d’un accès simultané. Sinon, vous pouvez copier temporairement l’objet blob sur plusieurs comptes de stockage pour améliorer le nombre total d’E/S par objet blob et sur les comptes de stockage. Les résultats varient en fonction du comportement de votre application. Veillez donc à tester les modèles d’accès concurrentiel lors de la conception.

### <a name="bandwidth-and-operations-per-blob"></a>Bande passante et opérations par objet blob

Un seul objet blob prend en charge plus de 500 requêtes par seconde. Si vous risquez de dépasser ces limites lorsque plusieurs clients doivent lire le même objet blob, il est conseillé d’utiliser un compte de stockage d’objet blob de blocs. Un compte de stockage d’objets blob de blocs fournit un débit de requêtes plus élevé ou un nombre supérieur opérations d’E/S par seconde (IOPS).

Vous pouvez utiliser un réseau de distribution de contenu (CDN, Content Delivery Network) tel que le CDN Azure pour distribuer les opérations sur le blob. Pour plus d’informations sur le CDN Azure, consultez [Vue d’ensemble sur le CDN Azure](../../cdn/cdn-overview.md).  

## <a name="partitioning"></a>Partitionnement

L’amélioration des performances passe par la compréhension de la manière dont Stockage Azure partitionne vos données de blob. Stockage Azure peut traiter les données dans une seule partition plus rapidement que les données qui s’étendent sur plusieurs partitions. En nommant vos objets blob de manière appropriée, vous pouvez améliorer l’efficacité des requêtes de lecture.

Le stockage blob utilise un schéma de partitionnement basé sur une plage pour mettre à l’échelle et équilibrer la charge. Chaque objet blob possède une clé de partition composée du nom d’objet blob complet (compte+conteneur+objet blob). La clé de partition est utilisée pour partitionner les données blob en plages. La charge des plages est ensuite équilibrée dans le stockage d’objets blob.

Le partitionnement basé sur une plage signifie que les conventions d’affectation de noms qui utilisent l’ordonnancement lexical (par exemple, *mypayroll* , *myperformance* , *myemployees* , etc.) ou des horodatages (*log20160101* , *log20160102* , *log20160102*, etc.) sont plus susceptibles d’entraîner la colocalisation des partitions sur le même serveur de partition, jusqu’à ce que l’augmentation de la charge exige qu’elles soient divisées en plages plus petites. La colocalisation des objets blob sur le même serveur de partition améliore les performances ; par conséquent, l’amélioration des performances implique en grande partie l’attribution d’un nom aux objets blob de manière à les organiser plus efficacement.

Par exemple, tous les objets blob d’un conteneur peuvent être fournis par un même serveur jusqu'à ce que la charge sur ces objets blob nécessite un rééquilibrage plus approfondi des plages de la partition. De même, un groupe de comptes faiblement chargés avec leurs noms classés par ordre lexical peut être pris en charge par un même serveur jusqu'à ce que la charge sur un ou tous ces comptes nécessite qu’ils soient répartis entre plusieurs serveurs de partitions.

Chaque opération d’équilibrage de charge peut affecter la latence des appels de stockage lors de l’opération. La capacité du service à gérer une hausse soudaine du trafic vers une partition est limitée par la scalabilité d’un serveur à partition unique jusqu’à ce que l’opération d’équilibrage de charge intervienne et rééquilibre la plage clé de la partition.

Vous pouvez suivre quelques bonnes pratiques pour réduire la fréquence de ces opérations.  

- Si possible, utilisez des tailles d’objet blob ou de bloc supérieures à 4 Mio pour les comptes de stockage standard et supérieures à 256 Kio pour les comptes de stockage Premium. Les tailles d’objet blob ou de bloc supérieures activent automatiquement les objets blob de blocs à haut débit. L’objet blob de blocs à haut débit fournit une ingestion haute performance qui n’est pas affectée par l’affectation de noms aux partitions.
- Examinez la convention de nommage que vous utilisez pour les comptes, conteneurs, objets blob, tables et files d’attente. Vous pouvez ajouter un préfixe aux noms de comptes, de conteneurs ou d’objets blob avec un hachage à trois chiffres à l’aide d’une fonction de hachage qui correspond le mieux à vos besoins.
- Si vous organisez vos données à l'aide d'horodatages ou d’identificateurs numériques, veillez à ne pas utiliser un modèle de trafic « Ajouter après uniquement » ou « Ajouter avant uniquement ». Ces modèles ne conviennent pas à un système de partitionnement basé sur une plage. Ces modèles peuvent entraîner tout le trafic vers une partition unique et limiter l’équilibrage de charge du système.

    Par exemple, si vos opérations quotidiennes utilisent un objet blob avec un horodatage comme *aaaammjj*, tout le trafic pour cette opération quotidienne est alors dirigé vers un objet blob unique pris en charge par un serveur à partition unique. Vérifiez si les limites par objet blob et par partition répondent à vos besoins, et pensez à diviser cette opération en plusieurs objets blob si nécessaire. De même, si vous stockez des données chronologiques dans vos tables, tout le trafic peut être dirigé vers la dernière partie de l’espace de noms de clé. Si vous utilisez des ID numériques, préfixez l’ID avec un hachage à trois chiffres. Si vous utilisez des horodatages, préfixez l’horodatage avec la valeur en secondes, par exemple *ssaaaammjj* . Si votre application effectue régulièrement des opérations de listage et d’interrogation, choisissez une fonction de hachage qui limite le nombre de vos requêtes. Dans certains cas, un préfixe aléatoire peut être suffisant.
  
- Pour plus d’informations sur le schéma de partitionnement utilisé dans le Stockage Azure, consultez [Stockage Azure : un service de stockage cloud hautement disponible à cohérence forte](https://sigops.org/sosp/sosp11/current/2011-Cascais/printable/11-calder.pdf).

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

Pour une distribution étendue de contenu d’objets blob, utilisez un réseau de diffusion de contenu tel que le CDN Azure. Pour plus d’informations sur le CDN Azure, voir la page [CDN Azure](../../cdn/cdn-overview.md).  

## <a name="sas-and-cors"></a>SAP et CORS

Supposons que vous deviez autoriser du code, tel que du code JavaScript qui s’exécute dans le navigateur web d’un utilisateur ou dans une application de téléphone mobile, à accéder aux données du stockage Azure. L’une des méthodes possibles consiste à créer une application de service qui serve de proxy. L’appareil de l’utilisateur s’authentifie auprès du service, qui à son tour autorise l’accès aux ressources du stockage Azure. Vous évitez ainsi d’exposer vos clés de compte de stockage sur des appareils non sécurisés. Cependant, cette méthode entraîne une surcharge importante pour l’application du service, dans la mesure où toutes les données transférées entre l’appareil de l’utilisateur et le stockage Azure doivent transiter par l’application du service.

Vous pouvez éviter d’utiliser une application de service en tant que proxy pour le stockage Azure en utilisant des signatures d’accès partagé (SAS). Avec les signatures d’accès partagé, vous pouvez autoriser l’appareil de votre utilisateur à adresser directement des requêtes au stockage Azure par le biais d’un jeton à accès limité. Par exemple, si un utilisateur souhaite charger une photo vers votre application, votre application de service peut générer une signature d’accès partagé et l’envoyer à l’appareil de l’utilisateur. Le jeton SAS peut accorder l’autorisation d’écrire dans une ressource du stockage Azure pendant un intervalle de temps spécifié, au bout duquel le jeton SAS expire. Pour plus d’informations sur les SAS, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAS)](../common/storage-sas-overview.md).  

En règle générale, un navigateur web n’autorise pas le code JavaScript d’une page hébergée par un site web d’un domaine à effectuer certaines opérations, telles que les opérations d’écriture, sur un autre domaine. Connue sous le nom de stratégie de même origine, cette stratégie empêche un script malveillant d’une page d’obtenir l’accès aux données d’une autre page web. Toutefois, la stratégie de même origine peut représenter un obstacle lorsque vous créez une solution dans le cloud. Le partage des ressources cross-origin (CORS) est une fonctionnalité de navigateur qui autorise le domaine cible à indiquer au navigateur que vous approuvez les requêtes en provenance du domaine source.

Supposons, par exemple, qu’une application web exécutée dans Azure envoie une requête pour une ressource à un compte de stockage Azure. L’application web est le domaine source, et le compte de stockage est le domaine cible. Vous pouvez configurer le partage des ressources cross-origin pour l’un des services du stockage Azure afin d’indiquer au navigateur web que les requêtes provenant du domaine source sont approuvées par le stockage Azure. Pour plus d’informations sur le partage des ressources cross-origin, consultez [Prise en charge du partage des ressources cross-origin (CORS) pour le stockage Azure](/rest/api/storageservices/Cross-Origin-Resource-Sharing--CORS--Support-for-the-Azure-Storage-Services).  
  
Les signatures d’accès partagé et le partage des ressources cross-origin vous aident à éviter une charge inutile sur votre application web.  

## <a name="caching"></a>Mise en cache

La mise en cache joue un rôle essentiel dans les performances. Les sections suivantes abordent les meilleures pratiques en matière de mise en cache.

### <a name="reading-data"></a>Lecture des données en cours

En général, il est préférable de lire les données deux fois. Prenons l’exemple d’une application Web qui a récupéré un objet blob de 50 Mio à partir de Stockage Azure pour servir de contenu à un utilisateur. Dans l’idéal, l’application met en cache l’objet blob localement sur le disque, puis récupère la version mise en cache pour les requêtes ultérieures de l’utilisateur.

Un moyen d’éviter de récupérer un blob s’il n'a pas été modifié depuis sa mise en cache est de qualifier l’opération GET avec un en-tête conditionnel pour l’heure de modification. Si l’heure de la dernière modification est postérieure à l’heure à laquelle l’objet blob a été mis en cache, l’objet blob est récupéré et mis en cache de nouveau. Dans le cas contraire, l’objet blob mis en cache est récupéré pour des performances optimales.

Vous pouvez également décider de concevoir votre application pour qu’elle suppose que l’objet blob reste inchangé pendant une brève période après récupération. Dans ce cas, l’application n’a pas besoin de vérifier si l’objet blob a été modifié au cours de cet intervalle.

Les données de configuration, les données de recherche et d’autres données fréquemment utilisées par l’application constituent de parfaits candidats pour la mise en cache.  

Pour plus d’informations sur les en-têtes conditionnels, consultez [Spécification des en-têtes conditionnels pour les opérations du service Blob](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations).  

### <a name="uploading-data-in-batches"></a>Chargement de données par lots

Dans certains scénarios, vous pouvez agréger des données en local, puis les charger périodiquement dans un lot au lieu de les télécharger immédiatement une à une. Par exemple, supposons qu’une application web conserve un fichier journal d’activités. L’application peut soit charger, sous la forme d’une entité de table, les détails de chaque activité lorsqu’elle se produit (ce qui implique de nombreuses opérations de stockage), soit enregistrer les détails de l’activité dans un fichier journal local, puis les charger régulièrement vers un objet blob sous la forme d’un fichier délimité. Si chaque entrée du journal a une taille de 1 Ko, vous pouvez charger des milliers d’entrées au cours d’une même transaction. Une même transaction prend en charge le chargement d’un objet blob d’une taille maximale de 64 Mio. Le développeur d’applications doit tenir compte du risque d’échecs de chargement ou de l’appareil client. Si les données d’activité doivent être chargées durant un intervalle de temps plutôt que dans le cadre d’une seule activité, il est recommandé d’utiliser le stockage d’objets blob sur le stockage Table.

## <a name="net-configuration"></a>Configuration .NET

Si vous utilisez .NET Framework, vous trouverez dans cette section plusieurs paramètres de configuration rapide que vous pouvez appliquer pour améliorer sensiblement les performances.  Si vous utilisez un autre langage, vérifiez si des concepts similaires y sont associés.  

### <a name="use-net-core"></a>Utiliser .NET Core

Développez vos applications du stockage Azure avec .NET Core 2.1 ou ultérieur pour tirer parti des performances améliorées. L’utilisation de .NET Core 3.x est recommandée dans la mesure du possible.

Pour plus d’informations sur les améliorations des performances dans .NET Core, consultez les billets de blog suivants :

- [Performance Improvements in .NET Core 3.0](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-3-0/)
- [Performance Improvements in .NET Core 2.1](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-2-1/)

### <a name="increase-default-connection-limit"></a>Augmenter la limite de connexions par défaut

Dans .NET, le code suivant augmente la limite de connexions par défaut (qui est généralement de 2 dans un environnement client ou de 10 dans un environnement de serveur) à 100. En règle générale, vous devez définir la valeur sur environ le nombre de threads utilisés par votre application. Définissez la limite de connexions avant d’ouvrir une connexion.

```csharp
ServicePointManager.DefaultConnectionLimit = 100; //(Or More)  
```

Pour les autres langages de programmation, voir la documentation pour savoir comment définir la limite de connexions.  

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

## <a name="copying-and-moving-blobs"></a>Copie et déplacement d’objets blob

Le stockage Azure fournit un certain nombre de solutions pour copier et déplacer des objets blob dans un compte de stockage, entre des comptes de stockage, et entre des systèmes locaux et le cloud. Cette section décrit certaines de ces options en terme d’effets sur les performances. Pour plus d’informations sur le transfert efficace de données vers ou depuis le stockage d’objets blob, consultez [Choisir une solution Azure pour le transfert de données](../common/storage-choose-data-transfer-solution.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

### <a name="blob-copy-apis"></a>API de copie d’objets blob

Pour copier des objets blob entre des comptes de stockage, utilisez l’opération [Put Block from URL](/rest/api/storageservices/put-block-from-url) (Placer un bloc à partir d’une URL). Cette opération copie les données de façon synchrone à partir de n’importe quelle source d’URL dans un objet blob de blocs. L’opération `Put Block from URL` permet de réduire considérablement la bande passante requise lorsque vous migrez des données entre des comptes de stockage. Étant donné que l’opération de copie a lieu côté service, vous n’avez pas besoin de télécharger et de charger à nouveau les données.

Pour copier des données dans le même compte de stockage, utilisez l’opération [Copier l’objet blob](/rest/api/storageservices/Copy-Blob). Les copies de données effectuées au sein du même compte de stockage sont généralement plus rapides.  

### <a name="use-azcopy"></a>Utiliser AzCopy

L’utilitaire de ligne de commande AzCopy est une option simple et efficace pour le transfert en bloc d’objets blob vers, à partir de et entre les comptes de stockage. AzCopy est optimisé pour ce scénario et peut générer des taux de transfert élevés. AzCopy version 10 utilise l’opération `Put Block From URL` pour copier des données d’objets blob entre des comptes de stockage. Pour plus d’informations, consultez [Copier ou déplacer des données vers Stockage Azure avec AzCopy v10](../common/storage-use-azcopy-v10.md).  

### <a name="use-azure-data-box"></a>Utiliser Azure Data Box

Pour importer des volumes importants de données dans le stockage d’objets blob, envisagez d’utiliser la famille de produits Azure Data Box pour les transferts hors connexion. Les appareils Data Box fournis par Microsoft constitue une bonne solution pour transférer les volumes de données importants vers Azure si vous êtes limité par le temps, la disponibilité du réseau ou les coûts. Pour plus d’informations, consultez la [documentation Azure DataBox](../../databox/index.yml).

## <a name="content-distribution"></a>Distribution de contenu

Il arrive qu’une application doive diffuser le même contenu vers plusieurs utilisateurs situés au sein d’une même région ou dans des régions différentes. Il peut s’agir, par exemple, d’une vidéo de démonstration d’un produit utilisée sur la page d’accueil d’un site web. Dans ce scénario, utilisez un réseau de distribution de contenu (CDN, Content Delivery Network) comme le CDN Azure pour distribuer le contenu d’objets blob au niveau géographique. Contrairement à un compte Azure Storage qui existe dans une seule région et qui ne peut pas diffuser de contenu avec une faible latence vers d’autres régions, le CDN Azure utilise des serveurs dans plusieurs centres de données répartis dans le monde entier. De plus, un CDN peut généralement prendre en charge des limites de sortie bien plus élevées qu’un compte de stockage unique.  

Pour plus d’informations sur le CDN Azure, voir la page [CDN Azure](../../cdn/cdn-overview.md).

## <a name="use-metadata"></a>Utiliser les métadonnées

Le service BLOB prend en charge les requêtes HEAD, qui peuvent inclure des métadonnées ou des propriétés d’objet blob. Par exemple, si votre application a besoin de données EXIF (format d’image modifiable) d’une photo, elle peut récupérer la photo et l’extraire. Pour économiser de la bande passante et améliorer les performances, votre application peut stocker les données EXIF dans les métadonnées de l’objet BLOB lorsque l’application charge la photo. Vous pouvez ensuite récupérer les données EXIF dans les métadonnées à l’aide d’une requête HEAD uniquement. L’extraction des métadonnées uniquement et non du contenu total de l’objet BLOB permet d’économiser de la bande passante et de réduire le temps de traitement requis pour extraire les données EXIF. Gardez à l’esprit que chaque objet blob peut stocker 8 Kio de métadonnées.  

## <a name="upload-blobs-quickly"></a>Charger des objets blob rapidement

Pour charger des objets blob rapidement, commencez par déterminer si vous chargez un ou plusieurs objets blob. Utilisez les instructions ci-dessous pour déterminer la méthode correcte en fonction de votre scénario.  

### <a name="upload-one-large-blob-quickly"></a>Charger rapidement un objet blob volumineux

Pour télécharger rapidement un seul objet blob de grande taille, une application cliente peut télécharger ses blocs ou pages en parallèle en tenant compte des objectifs d’extensibilité des différents objets blob et du compte de stockage dans son ensemble. Les bibliothèques clientes de Stockage Microsoft Azure prennent en charge le chargement en parallèle. Par exemple, vous pouvez utiliser les propriétés suivantes pour spécifier le nombre de requêtes simultanées autorisées dans .NET ou Java. Les bibliothèques clientes fournissent des options similaires pour d’autres langages pris en charge.

- Pour .NET, définissez la propriété [BlobRequestOptions.ParallelOperationThreadCount](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.paralleloperationthreadcount).
- Pour Java/Android, appelez la méthode [BlobRequestOptions.setConcurrentRequestCount(final Integer concurrentRequestCount)](/java/api/com.microsoft.azure.storage.blob.blobrequestoptions.setconcurrentrequestcount).

### <a name="upload-many-blobs-quickly"></a>Charger rapidement de nombreux objets BLOB

Pour télécharger rapidement de nombreux objets blob, effectuez cette opération en parallèle. Le chargement parallèle s’avère plus rapide que le chargement d’objets blob individuels avec des chargements de blocs parallèles, dans la mesure où le transfert est réparti entre plusieurs partitions du service de stockage. AzCopy effectue des téléchargements en parallèle et son utilisation est recommandée pour ce scénario. Pour plus d’informations, consultez [Bien démarrer avec AzCopy](../common/storage-use-azcopy-v10.md).  

## <a name="choose-the-correct-type-of-blob"></a>Choisir le type d’objet blob approprié

Stockage Microsoft Azure prend en charge les objets blob de blocs, d’ajout et de pages. Pour un scénario d’utilisation donné, le type d’objet blob choisi affecte les performances et l’extensibilité de la solution.

Les objets blob de blocs sont appropriés lorsque vous souhaitez charger efficacement de grandes quantités de données. Par exemple, une application cliente qui charge des photos ou des vidéos vers le stockage d’objets blob ciblerait des objets blob de blocs.

Les objets blob d’ajout sont similaires aux objets blob de blocs, car ils sont composés de blocs. Lorsque vous modifiez un objet blob d’ajout, les blocs sont ajoutés à la fin de l’objet blob uniquement. Les objets blob d’ajout sont utiles dans des scénarios tels que la journalisation, lorsqu’une application doit ajouter des données à un objet blob existant.

Les objets blob de pages sont appropriées si l’application doit effectuer des écritures aléatoires sur les données. Par exemple, les disques durs virtuels d’Azure sont stockés en tant qu’objets blob de pages. Pour plus d’informations, consultez [Présentation des objets blob de blocs, des objets blob d’ajout et des objets blob de pages](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs).  

## <a name="next-steps"></a>Étapes suivantes

- [Objectifs de performance et d’extensibilité du Stockage Blob](scalability-targets.md)
- [Objectifs d’extensibilité et de performances pour les comptes de stockage standard](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Codes d’état et d’erreur](/rest/api/storageservices/Status-and-Error-Codes2)