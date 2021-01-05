---
title: Technologies en mémoire
description: Les technologies en mémoire améliorent considérablement les performances des charges de travail transactionnelles et analytiques d’Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: ''
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/19/2019
ms.openlocfilehash: 48b74a5507eb4a1d48b7bf70133e476a30fe8169
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92779949"
---
# <a name="optimize-performance-by-using-in-memory-technologies-in-azure-sql-database-and-azure-sql-managed-instance"></a>Optimiser les performances en utilisant les technologies en mémoire d’Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Les technologies en mémoire vous permettent d’améliorer les performances de votre application, et potentiellement de réduire le coût de votre base de données.

## <a name="when-to-use-in-memory-technologies"></a>Quand utiliser les technologies en mémoire

Grâce aux technologies en mémoire, vous pouvez améliorer vos performances au niveau des charges de travail suivantes :

- **Transactionnelle** (traitement transactionnel en ligne (OLTP)) où la plupart des demandes lisent ou mettent à jour un plus petit jeu de données (par exemple, les opérations CRUD).
- **Analytique** (traitement analytique en ligne (OLAP)) où la plupart des requêtes ont des calculs complexes pour la création de rapports, avec un certain nombre de requêtes qui chargent et ajoutent des données aux tables existantes (chargement en masse) ou suppriment les données des tables.
- **Mixte** (traitement transactionnel/analytique hybride (HTAP)) où les requêtes OLTP et OLAP sont exécutées sur le même jeu de données.

Les technologies en mémoire peuvent améliorer les performances de ces charges de travail en conservant dans la mémoire les données à traiter, en utilisant la compilation native des requêtes ou un traitement avancé comme le traitement par lots et des instructions SIMD qui sont disponibles sur le matériel sous-jacent.

## <a name="overview"></a>Vue d’ensemble

Azure SQL Database et Azure SQL Managed Instance disposent des technologies en mémoire suivantes :

- *[OLTP en mémoire](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)* augmente le nombre de transactions par seconde et réduit la latence du traitement transactionnel. Les scénarios qui bénéficient de l’OLTP en mémoire sont : le traitement de transactions haut débit, notamment les données commerciales et de jeux, l’ingestion de données d’événements ou d’appareils IoT, la mise en cache, le chargement de données, les tables temporaires et les scénarios de variables de table.
- Les *index columnstore en cluster* réduisent l’encombrement de stockage (jusqu'à 10 fois) et améliorent les performances des requêtes d’analyse et de création de rapports. Vous pouvez les utiliser avec des tables de faits dans vos mini-Data Warehouses pour faire tenir plus de données dans votre base de données et optimiser les performances. Vous pouvez également les utiliser avec des données historiques dans votre base de données opérationnelles pour archiver et être en mesure d’interroger jusqu’à 10 fois plus de données.
- Les *index columnstore sans cluster* pour HTAP vous aident à obtenir un aperçu en temps réel de votre activité en interrogeant la base de données opérationnelle directement, sans avoir à exécuter de processus d’extraction, de transformation et de chargement (ETL) coûteux et à attendre que l’entrepôt de données se remplisse. Les index columnstore sans cluster permettent une exécution rapide des requêtes d’analyse sur la base de données OLTP, tout en réduisant l’impact sur la charge de travail opérationnelle.
- Les *index columnstore en cluster à mémoire optimisée* pour HTAP vous permettent d’effectuer un traitement transactionnel rapide et d’exécuter *simultanément* et très rapidement des requêtes analytiques sur les mêmes données.

Les index columnstore et la technologie OLTP en mémoire sont inclus dans le produit SQL Server respectivement depuis 2012 et 2014. Azure SQL Database, Azure SQL Managed Instance et SQL Server partagent la même implémentation pour les technologies en mémoire.

## <a name="benefits-of-in-memory-technology"></a>Avantages de la technologie en mémoire

Les technologies en mémoire vous aident également à réduire les coûts grâce à un traitement plus efficace des requêtes et des transactions. En général, il n’est pas nécessaire de mettre à jour le niveau de tarification de la base de données pour obtenir des gains de performances. Dans certains cas, vous pourriez même être en mesure de réduire le niveau de tarification, tout en bénéficiant d’une amélioration des performances grâce aux technologies en mémoire.

Voici deux exemples illustrant comment la technologie OLTP en mémoire a permis d’améliorer les performances :

- À l’aide de l’OLTP en mémoire, [Quorum Business Solutions a pu doubler leur charge de travail tout en améliorant les DTU de 70 %](https://resources.quorumsoftware.com/case-studies/quorum-doubles-key-database-s-workload-while-lowering-dtu).
- La vidéo suivante montre une amélioration significative de la consommation de ressources avec un exemple de charge de travail : [Vidéo sur l'OLTP en mémoire](https://channel9.msdn.com/Shows/Data-Exposed/In-Memory-OTLP-in-Azure-SQL-DB) Pour plus d’informations, consultez le billet de blog : [OLTP en mémoire](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/)

> [!NOTE]  
> Les technologies en mémoire sont disponibles aux niveaux Premium et Critique pour l’entreprise.

La vidéo suivante explique les gains de performance potentiels que peuvent apporter les technologies en mémoire. N’oubliez pas que le gain de performance que vous remarquez dépend de nombreux facteurs, notamment de la nature de la charge de travail et des données, du modèle d’accès de la base de données, etc.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-In-Memory-Technologies/player]
>

Cet article décrit des aspects de l'OLTP en mémoire et des index columnstore qui sont spécifiques à Azure SQL Database et Azure SQL Managed Instance, et inclut également des exemples :

- Vous verrez l’impact de ces technologies sur le stockage et les limites de taille des données.
- Vous verrez ensuite comment gérer le déplacement de bases de données qui exploitent ces technologies entre les différents niveaux tarifaires.
- Vous verrez deux exemples qui illustrent l'utilisation de l'OLTP en mémoire, ainsi que les index columnstore.

Pour plus d’informations sur les technologies en mémoire dans SQL Server, consultez :

- [Présentation de l’OLTP en mémoire et scénarios d’utilisation](/sql/relational-databases/in-memory-oltp/overview-and-usage-scenarios) (avec notamment des références à des études de cas client et des informations de prise en main)
- [Documentation pour l’OLTP In-Memory](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)
- [Description des index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview)
- Traitement hybride transactionnel et analytique (HTAP), également appelé [analytique opérationnelle en temps réel](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)

## <a name="in-memory-oltp"></a>OLTP en mémoire

La technologie OLTP en mémoire fournit des opérations d’accès aux données extrêmement rapides en conservant toutes les données en mémoire. Elle utilise également des index spécialisés, la compilation native des requêtes et un accès aux données sans verrou pour améliorer les performances de la charge de travail OLTP. Il existe deux façons d’organiser vos données OLTP en mémoire :

- Le format **rowstore à mémoire optimisée** où chaque ligne est un objet de mémoire distinct. Il s’agit d’un format OLTP en mémoire classique optimisé pour les charges de travail OLTP hautes performances. Il existe deux types de tables à mémoire optimisée qui peuvent être utilisées dans le format rowstore à mémoire optimisée :

  - *Tables durables* (SCHEMA_AND_DATA) où les lignes placées en mémoire sont conservées après le redémarrage du serveur. Ce type de tables se comporte comme une table rowstore traditionnelle avec en plus les avantages des optimisations en mémoire.
  - *Tables non durables* (SCHEMA_ONLY) où les lignes ne sont pas conservées après le redémarrage. Ce type de table est conçu pour les données temporaires (par exemple, le remplacement de tables temporaires) ou les tables dans lesquelles vous devez rapidement charger des données avant de les déplacer vers une table persistante (les tables de mise en lots).

- Le format **columnstore à mémoire optimisée** où les données sont organisées dans un format de colonne. Cette structure est conçue pour les scénarios HTAP où vous avez besoin d’exécuter des requêtes analytiques sur la même structure de données que celle où votre charge de travail OLTP s’exécute.

> [!Note]
> La technologie OLTP en mémoire est conçue pour les structures de données qui peuvent résider entièrement en mémoire. Comme les données en mémoire ne peuvent pas être déchargées sur le disque, utilisez une base de données avec suffisamment de mémoire. Consultez [Limite de la taille des données et du stockage pour OLTP en mémoire](#data-size-and-storage-cap-for-in-memory-oltp) pour plus d’informations.

Notions fondamentales sur OLTP en mémoire : [Démarrage rapide 1 : Technologies OLTP en mémoire pour des performances T-SQL plus rapides](/sql/relational-databases/in-memory-oltp/survey-of-initial-areas-in-in-memory-oltp) (un autre article pour vous aider à commencer)

Vidéos détaillées sur les technologies :

- [OLTP en mémoire](https://channel9.msdn.com/Shows/Data-Exposed/In-Memory-OTLP-in-Azure-SQL-DB) (avec une démonstration des avantages en termes de performances et les étapes à suivre pour reproduire ces résultats vous-même)
- [Vidéos sur OLTP en mémoire : Qu’est-ce-que c’est, et quand/comment l’utiliser](/archive/blogs/sqlserverstorageengine/in-memory-oltp-video-what-it-is-and-whenhow-to-use-it)

Vous pouvez comprendre par programmation si une base de données spécifique prend en charge l’OLTP en mémoire. Vous pouvez exécuter la requête Transact-SQL suivante :

```sql
SELECT DatabasePropertyEx(DB_NAME(), 'IsXTPSupported');
```

Si la requête renvoie **1** , l’OLTP en mémoire est pris en charge dans cette base de données. Les requêtes suivantes identifient tous les objets à supprimer avant qu'une base de données puisse être rétrogradée au niveau Usage général, Standard ou De base :

```sql
SELECT * FROM sys.tables WHERE is_memory_optimized=1
SELECT * FROM sys.table_types WHERE is_memory_optimized=1
SELECT * FROM sys.sql_modules WHERE uses_native_compilation=1
```

### <a name="data-size-and-storage-cap-for-in-memory-oltp"></a>Seuil de la taille des données et du stockage pour l’OLTP en mémoire

l’OLTP en mémoire inclut des tables optimisées en mémoire, qui sont utilisées pour stocker des données de l’utilisateur. Le volume de ces tables doit tenir dans la mémoire. Étant donné que vous gérez la mémoire directement dans SQL Database, nous disposons du concept de quota pour les données utilisateur. Ce concept est appelé *stockage OLTP en mémoire* .

Chaque niveau tarifaire de base de données unique pris en charge et chaque niveau tarifaire de pool élastique inclut une certaine quantité de stockage OLTP en mémoire.

- [Limites de ressources basées sur des unités DTU - Base de données unique](database/resource-limits-dtu-single-databases.md)
- [Limites de ressources basées sur des unités DTU - Pools élastiques](database/resource-limits-dtu-elastic-pools.md)
- [Limites de modèle de ressources basées sur des vCore : base de données unique](database/resource-limits-vcore-single-databases.md)
- [Limites de ressources basées sur des vCore : pools élastiques](database/resource-limits-vcore-elastic-pools.md)
- [Limites de ressources basées sur des vCore - instance managée](managed-instance/resource-limits.md)

Les éléments suivants sont pris en compte dans votre plafond de stockage OLTP en mémoire :

- Lignes de données utilisateur actives dans des tables optimisées en mémoire et variables de table. Notez que les anciennes versions des lignes ne comptent pas dans le seuil.
- Index de tables optimisées en mémoire.
- Coûts de fonctionnement des opérations ALTER TABLE.

Si vous atteignez le seuil, vous recevrez une erreur de quota et vous ne serez plus en mesure d’insérer ou de mettre à jour des données. Pour atténuer cette erreur, supprimez des données ou augmentez le niveau tarifaire de la base de données ou du pool.

Pour plus d’informations sur la surveillance de l’utilisation du stockage OLTP en mémoire et la configuration des alertes lorsque le seuil est presque atteint, consultez [Surveiller le stockage en mémoire](in-memory-oltp-monitor-space.md).

#### <a name="about-elastic-pools"></a>À propos des pools élastiques

Avec les pools élastiques, le stockage OLTP en mémoire est partagé entre toutes les bases de données dans le pool. Par conséquent, l’utilisation dans une base de données peut potentiellement affecter les autres bases de données. Voici deux solutions :

- Configurez une valeur `Max-eDTU` ou `MaxvCore` pour les bases de données inférieur au nombre d’eDTU ou de vCores du pool dans son ensemble. Cette valeur maximale limite l’utilisation du stockage OLTP en mémoire, dans n’importe quelle base de données du pool, à la taille qui correspond au nombre d’eDTU.
- Configurez une valeur `Min-eDTU` ou `MinvCore` supérieure à 0. Cette valeur minimale garantit que chaque base de données dans le pool possède la quantité de stockage OLTP en mémoire disponible qui correspond à l’élément `Min-eDTU` ou `vCore` configuré.

### <a name="changing-service-tiers-of-databases-that-use-in-memory-oltp-technologies"></a>Changement des niveaux de service des bases de données qui utilisent des technologies OLTP en mémoire

Vous pouvez toujours mettre à niveau votre base de données ou instance vers un niveau supérieur, comme passer du niveau Usage général au niveau Critique pour l’entreprise (ou de Standard à Premium). Les ressources et les fonctionnalités disponibles ne font qu’augmenter.

Toutefois, le passage à un niveau inférieur peut impacter négativement votre base de données. L’impact est particulièrement évident quand vous descendez du niveau Critique pour l’entreprise au niveau Usage général (ou de Premium à Standard ou De base) et que votre base de données contient des objets OLTP en mémoire. Les tables optimisées par la mémoire ne sont pas disponibles après la mise à niveau (même si elles restent visibles). Les mêmes considérations s’appliquent lors de la réduction du niveau tarifaire d’un pool élastique ou du déplacement d’une base de données avec des technologies en mémoire vers un pool élastique de niveau Usage général, Standard ou De base.

> [!Important]
> OLTP en mémoire n’est pas pris en charge dans les niveaux Usage général, Standard ou De base. Vous ne pouvez donc pas déplacer une base de données qui contient des objets OLTP en mémoire vers un de ces niveaux.

Avant de rétrograder la base de données vers le niveau Usage général, Standard ou De base, supprimez toutes les tables à mémoire optimisée et tous les types de table, ainsi que tous les modules T-SQL compilés en mode natif.

*Effectuer un scale-down des ressources dans le niveau Critique pour l’entreprise*  : Les données des tables à mémoire optimisée doivent tenir dans le stockage OLTP en mémoire qui est associé au niveau de la base de données ou de l’instance gérée, ou qui est disponible dans le pool élastique. Si vous essayez d’effectuer un scale-down du niveau ou de déplacer la base de données dans un pool qui n’a pas de stockage OLTP en mémoire suffisant, l’opération échoue.

## <a name="in-memory-columnstore"></a>Columnstore en mémoire

La technologie columnstore en mémoire vous offre la possibilité de stocker et d’interroger une grande quantité de données dans les tables. La technologie columnstore utilise le format de stockage de données en colonnes et le traitement de requête par lots pour obtenir jusqu’à 10 fois les performances de requêtes dans les charges de travail OLAP par rapport au stockage traditionnel en lignes. Vous pouvez également obtenir jusqu’à 10 fois la compression de données par rapport à la taille des données décompressées.
Il existe deux types de modèles columnstore pour organiser vos données :

- **Columnstore en cluster** où toutes les données de la table sont organisées en colonnes. Dans ce modèle, toutes les lignes de la table sont placées sous forme de colonnes qui compressent fortement les données et vous permet d’exécuter des requêtes analytiques et des rapports rapides sur la table. Selon la nature de vos données, leur taille peut être réduite de 10 à 100 fois. Le modèle columnstore en cluster permet également une ingestion rapide de grandes quantités de données (chargement en masse), grâce à la compression des gros lots de données supérieurs à 100 000 lignes avant leur stockage sur le disque. Choisissez ce modèle pour les scénarios d’entrepôt de données classiques.
- **Columnstore non-cluster** où les données sont stockées dans une table rowstore traditionnelle et où un index au format columnstore est utilisé pour les requêtes analytiques. Ce modèle permet un traitement transactionnel-analytique hybride (HTAP) : la possibilité d’exécuter une analytique performante en temps réel sur une charge de travail transactionnelle. Les requêtes OLTP sont exécutées sur la table rowstore qui est optimisée pour l’accès à un petit ensemble de lignes, tandis que les requêtes OLAP sont exécutées sur l’index columnstore qui est le meilleur choix pour les analyses et l’analytique. L'optimiseur de requête choisit dynamiquement le format rowstore ou columnstore en fonction de la requête. Les index columnstore non-cluster ne réduisent pas la taille des données, car le jeu de données d’origine est conservé dans la table rowstore initiale sans aucun changement. Toutefois, la taille de l’index columnstore supplémentaire doit être inférieure à celle de l’index B-tree équivalent.

> [!Note]
> La technologie columnstore en mémoire conserve uniquement les données qui sont nécessaires pour le traitement dans la mémoire, tandis que les données qui ne tiennent pas dans la mémoire sont stockées sur disque. Par conséquent, la quantité de données dans les structures columnstore en mémoire peut dépasser la quantité de mémoire disponible.

Vidéo détaillée sur la technologie :

- [Index columnstore : Vidéos sur l’analytique en mémoire d’Ignite 2016](/archive/blogs/sqlserverstorageengine/columnstore-index-in-memory-analytics-i-e-columnstore-index-videos-from-ignite-2016)

### <a name="data-size-and-storage-for-columnstore-indexes"></a>Taille des données et stockage pour les index columnstore

Le volume des index columntore ne doit pas forcément tenir dans la mémoire. Par conséquent, le seul seuil de taille des index est la taille de base de données globale maximale décrite dans les articles [Modèle d’achat DTU](database/service-tiers-dtu.md) et [Modèle d’achat vCore](database/service-tiers-vcore.md).

Lors de l’utilisation d’index columnstore en cluster, la compression en colonnes est utilisée pour le stockage de table de base. Cette compression peut réduire considérablement l’encombrement de stockage des données utilisateur, ce qui signifie que vous pouvez entrer davantage de données dans la base de données. De plus, la compression peut être accrue d’avantage avec [la compression d’archivage en colonnes](/sql/relational-databases/data-compression/data-compression#using-columnstore-and-columnstore-archive-compression). Le taux de compression que vous pouvez obtenir dépend de la nature des données, mais une compression égale à 10 fois n’est pas rare.

Par exemple, si vous disposez d’une base de données avec une taille maximale de 1 téraoctet (To), et que vous atteignez une compression de 10 fois à l’aide d’index columntore, vous pouvez afficher un total de 10 To de données utilisateur dans la base de données.

Lors de l’utilisation d’index columnstore sans cluster, la table de base est toujours stockée au format rowstore traditionnel. Par conséquent, les économies en stockage ne sont pas aussi significatives qu’avec des index columnstore en cluster. Toutefois, si vous remplacez un nombre d’index sans cluster traditionnels par un index columntore unique, vous pouvez toujours voir une économie globale d’espace de stockage pour la table.

### <a name="changing-service-tiers-of-databases-containing-columnstore-indexes"></a>Changement de niveaux de service des bases de données contenant des index columnstore

Le *passage d’une base de données unique au niveau De base ou Standard* n’est peut-être pas possible si le niveau de votre cible est inférieur à S3. Les index columnstore sont pris en charge uniquement sur le niveau tarifaire Critique pour l’entreprise/Premium et sur le niveau Standard, S3 et supérieur, et non sur le niveau De base. Lors de la rétrogradation de votre base de données vers un niveau non pris en charge, votre index columnstore devient indisponible. Le système maintient l’index columnstore, mais il ne tire jamais profit de l’index. Si vous mettez à niveau ultérieurement vers un niveau pris en charge, l’index columnstore est immédiatement prêt à être exploité à nouveau.

Si vous disposez d’un index columnstore **en cluster** , la totalité de la table devient indisponible après le passage à une version antérieure. Par conséquent, nous vous recommandons d’annuler tous les index columnstore *en index* avant de rétrograder votre base de données vers un niveau non pris en charge.

> [!Note]
> SQL Managed Instance prend en charge les index columnstore dans tous les niveaux.

<a id="install_oltp_manuallink" name="install_oltp_manuallink"></a>

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide 1 : Technologies OLTP en mémoire pour des performances T-SQL plus rapides](/sql/relational-databases/in-memory-oltp/survey-of-initial-areas-in-in-memory-oltp)
- [Utiliser OLTP en mémoire dans une application SQL Azure existante](in-memory-oltp-configure.md)
- [Surveiller le stockage OLTP en mémoire](in-memory-oltp-monitor-space.md) pour l’OLTP en mémoire
- [Essayer les fonctionnalités en mémoire](in-memory-sample.md)

## <a name="additional-resources"></a>Ressources supplémentaires

### <a name="deeper-information"></a>Informations supplémentaires

- [Découvrez comment le Quorum double la charge de travail de la base de données clé tout en réduisant les DTU de 70 %, grâce à l’OLTP en mémoire dans la SQL Database](https://customers.microsoft.com/story/quorum-doubles-key-databases-workload-while-lowering-dtu-with-sql-database)
- [Billet de blog sur l'OLTP en mémoire](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/)
- [En savoir plus sur In-Memory OLTP](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)
- [En savoir plus sur les index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview)
- [En savoir plus sur l’analytique opérationnelle en temps réel](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- Consultez [Modèles de charge de travail courants et considérations relatives à la migration](/previous-versions/dn673538(v=msdn.10)) (qui décrit des modèles de charge de travail où l’OLTP en mémoire apporte généralement des gains de performances significatifs)

### <a name="application-design"></a>Conception des applications

- [OLTP en mémoire (optimisation en mémoire)](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)
- [Utiliser OLTP en mémoire dans une application SQL Azure existante](in-memory-oltp-configure.md)

### <a name="tools"></a>Outils

- [Azure portal](https://portal.azure.com/)
- [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms)
- [Outils SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt)