---
title: Résoudre des problèmes de performances d’Apache HBase sur Azure HDInsight
description: Instructions et conseils pour le réglage d’Apache HBase afin d’obtenir des performances optimales sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 09/24/2019
ms.openlocfilehash: 5be3f02a80524d9c4b633e1e34d581fc26bfd32d
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547892"
---
# <a name="troubleshoot-apache-hbase-performance-issues-on-azure-hdinsight"></a>Résoudre des problèmes de performances d’Apache HBase sur Azure HDInsight

Cet article présente des instructions et conseils pour le réglage d’Apache HBase en vue d’obtenir des performances optimales sur Azure HDInsight. La plupart de ces conseils dépendent de la charge de travail particulière et du modèle de lecture/écriture/analyse. Avant d’apporter des modifications à la configuration d’un environnement de production, testez-les bien.

## <a name="hbase-performance-insights"></a>Insights sur les performances HBase

Le principal goulot d’étranglement dans la plupart des charges de travail HBase est le journal WAL (write-ahead log). Cela affecte sérieusement les performances d’écriture. HDInsight HBase a un modèle de calcul de stockage séparé. Les données sont stockées à distance sur le stockage Azure, même si les machines virtuelles hébergent les serveurs de région. Jusqu’à récemment, le write-ahead logging (WAL) était également écrit dans le stockage Azure. Dans HDInsight, ce problème a aggravé ce goulot d’étranglement. La fonctionnalité [Écritures accélérées](./apache-hbase-accelerated-writes.md) est conçue pour résoudre ce problème. Elle permet d’écrire le WAL sur des disques managés SSD Azure Premium. Cela permet d’améliorer considérablement les performances et de faire face à de nombreux problèmes liés à certaines charges de travail nécessitant un nombre important d’écritures.

Pour améliorer les opérations de lecture, utilisez un [compte de stockage d’objets blob de blocs Azure Premium](https://azure.microsoft.com/blog/azure-premium-block-blob-storage-is-now-generally-available/) comme stockage distant. Cette option est possible uniquement si la fonctionnalité WAL (write-ahead logging) est activée.

## <a name="compaction"></a>Compactage

Le compactage est un autre goulot d’étranglement potentiel clairement identifié. Par défaut, le compactage majeur est désactivé sur les clusters HDInsight HBase. Le compactage est désactivé car, même s’il s’agit d’un processus gourmand en ressources, les clients peuvent décider entièrement de sa planification en fonction de leurs charges de travail. Par exemple, ils peuvent le planifier pendant les heures creuses. De plus, la localisation des données n’est pas un problème, car notre stockage est distant (stockage Azure) et non un système de fichiers HDFS local.

Les clients doivent planifier un compactage majeur à leur convenance. S’ils n’effectuent pas cette opération de maintenance, le compactage aura un impact négatif sur les performances de lecture à long terme.

Pour les opérations d’analyse, des latences moyennes sensiblement supérieures à 100 ms doivent être considérées comme préoccupantes. Vérifiez si le compactage principal a été planifié avec précision.

## <a name="apache-phoenix-workload"></a>Charge de travail Apache Phoenix

Répondre aux questions suivantes vous aidera à mieux comprendre votre charge de travail Apache Phoenix :

* Vos « lectures » se traduisent-elles toutes en analyses ?
    * Dans ce cas, quelles sont les caractéristiques de ces analyses ?
    * Avez-vous optimisé votre schéma de table Phoenix pour ces analyses, notamment l’indexation appropriée ?
* Avez-vous utilisé l’instruction `EXPLAIN` pour comprendre les plans de requête que vos « lectures » génèrent ?
* Vos écritures sont-elles de type « upsert-Selects » ?
    * Si c’est le cas, elles effectuent également des analyses. La latence attendue pour les moyennes d’analyses est d’environ 100 millisecondes, alors qu’elle est de 10 millisecondes dans HBase.  

## <a name="test-methodology-and-metrics-monitoring"></a>Méthodologie de test et supervision des métriques

Si vous utilisez des tests d’évaluation tels que Yahoo! Cloud Serving Benchmark, JMeter ou Pherf pour tester et corriger les performances, vérifiez que :

- Les ordinateurs clients ne se transforment pas en goulot d’étranglement. Pour ce faire, vérifiez l’utilisation du processeur sur les ordinateurs clients.

- Les configurations côté client, telles que le nombre de threads, sont paramétrées de façon à saturer la bande passante allouée au client.

- Les résultats des tests sont enregistrés précisément et systématiquement.

Si les performances de vos requêtes se sont mises à baisser soudainement, regardez si le code de votre application contient des bogues. Génère-t-il soudainement de grandes quantités de données non valides ? Si c’est le cas, cela peut augmenter les latences de lecture.

## <a name="migration-issues"></a>Problèmes de migration

Si vous opérez une migration vers Azure HDInsight, procédez de manière systématique et précise, de préférence à l’aide d’une automatisation. Évitez d’effectuer une migration manuelle. Assurez-vous que :

- Les attributs de table appropriés sont compris dans la migration. Les attributs peuvent inclure la compression, les filtres de Bloom, etc.

- Dans les tables Phoenix, les paramètres de salage doivent être correctement mappés à la nouvelle taille du cluster. Par exemple, le nombre de compartiments Salt doit être un multiple du nombre de nœuds Worker compris dans le cluster. De plus, vous devez utiliser un multiple qui est un facteur de la quantité de zones réactives.

## <a name="server-side-configuration-tunings"></a>Réglages de la configuration côté serveur

Dans HDInsight HBase, les fichiers HFiles sont stockés dans le stockage distant. Si un élément est absent du cache, le coût des lectures est supérieur à celui des systèmes locaux, car les données des systèmes locaux sont sauvegardées par des systèmes HDFS locaux. Dans la plupart des scénarios, une utilisation intelligente de caches HBase (caches de bloc et de compartiment) permet de contourner cette difficulté. Dans les cas où le problème n’est pas contourné, l’utilisation d’un compte d’objet blob de blocs Premium peut aider à résoudre ce problème. Le pilote Stockage Blob Azure dépend de certaines propriétés telles que `fs.azure.read.request.size` pour extraire des données de blocs en fonction du mode de lecture (séquentielle ou aléatoire). Il se peut donc qu’il y ait encore des instances dont les latences de lecture sont plus élevées. Des expériences empiriques ont montré que le fait de définir la taille de bloc de requête de lecture (`fs.azure.read.request.size`) sur 512 Ko et d’adapter la taille de bloc des tables HBase en conséquence produit les meilleurs résultats.

Pour la plupart des clusters de nœuds de grande taille, HDInsight HBase fournit `bucketcache` sous la forme d’un fichier présent sur un disque SSD Premium local attaché à la machine virtuelle qui exécute `regionservers`. L’utilisation d’un cache off-heap peut apporter une certaine amélioration. Cette solution de contournement est limitée par l’utilisation de la mémoire disponible et le fait que la taille de celle-ci peut être inférieure à celle du cache basé sur un fichier. Il ne s’agit donc pas toujours du meilleur choix.

Voici quelques-uns des réglages de paramètres que nous avons effectués et qui semblent avoir aidé à différents niveaux :

- Augmentation de la taille de `memstore` par défaut de 128 Mo à 256 Mo. En règle générale, ce paramètre est recommandé pour les scénarios d’écriture intensive.

- Augmentation du nombre de threads qui sont dédiés au compactage par défaut (de **1** à **4** ). Ce paramètre est pertinent si nous observons de fréquents compactages mineurs.

- Évitement du blocage du vidage de `memstore` en raison d’une limite de magasin. Pour fournir cette mémoire tampon, définissez le paramètre `Hbase.hstore.blockingStoreFiles` sur **100** .

- Pour contrôler les vidages, utilisez les paramètres suivants :

    - `Hbase.regionserver.maxlogs`: **140** (évite les vidages en raison des limites WAL)

    - `Hbase.regionserver.global.memstore.lowerLimit`: **0,55**

    - `Hbase.regionserver.global.memstore.upperLimit`: **0,60**

- Configurations propres à Phoenix pour le réglage du pool de threads :

    - `Phoenix.query.queuesize`: **10000**

    - `Phoenix.query.threadpoolsize`: **512**

- Autres configurations propres à Phoenix :

    - `Phoenix.rpc.index.handler.count`: **50** (en cas de recherches d’index nombreuses ou volumineuses)

    - `Phoenix.stats.updateFrequency`: **1 heure**

    - `Phoenix.coprocessor.maxmetadatacachetimetolivems`: **1 heure**

    - `Phoenix.coprocessor.maxmetadatacachesize`: **50 Mo**

- Délais d’expiration RPC : **3 minutes**

   - Les délais d’expiration RPC incluent les délais d’expiration RPC HBase, les délais d’expiration des scanneurs clients HBase et les délais d’expiration des requêtes Phoenix. 
   - Vérifiez que le paramètre `hbase.client.scanner.caching` est défini sur la même valeur côté serveur et côté client. Si les valeurs ne sont pas les mêmes, ce paramètre provoque des erreurs côté client en lien avec `OutOfOrderScannerException`. Ce paramètre doit être défini sur une valeur basse pour les analyses volumineuses. Nous avons défini cette valeur sur **100** .

## <a name="other-considerations"></a>Autres considérations

Voici les paramètres supplémentaires à prendre en compte pour le paramétrage :

- `Hbase.rs.cacheblocksonwrite` : par défaut, ce paramètre est défini sur **true** dans HDI.

- Paramètres permettant de différer un compactage mineur.

- Paramètres expérimentaux tels que l’ajustement de pourcentages de files d’attente réservés aux requêtes de lecture et d’écriture.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème persiste, utilisez l’un des canaux suivants pour obtenir de l’aide :

- Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

- Contactez [@AzureSupport](https://twitter.com/azuresupport). Il s’agit du compte Microsoft Azure officiel pour améliorer l’expérience client. Il fournit à la communauté Azure les ressources appropriées : réponses, support technique et experts.

- Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Votre abonnement Microsoft Azure comprend l’accès au support relatif à la gestion et à la facturation des abonnements, et le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).