---
title: 'Architecture : Apache Hadoop en local vers Azure HDInsight'
description: Découvrez les bonnes pratiques concernant l’architecture pour la migration des clusters Hadoop locaux vers Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: ashishth
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/06/2019
ms.openlocfilehash: af9b1f42140c5656c5f55a98c2d635d59e130db5
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92533731"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---architecture-best-practices"></a>Migrer des clusters Apache Hadoop locaux vers Azure HDInsight - bonnes pratiques concernant l’architecture

Cet article fournit des recommandations relatives à l’architecture des systèmes Azure HDInsight. Il fait partie d’une série qui propose des bonnes pratiques pour aider à la migration des systèmes Apache Hadoop locaux vers Azure HDInsight.

## <a name="use-multiple-workload-optimized-clusters"></a>Utiliser plusieurs clusters de charge de travail optimisée

De nombreux déploiements Apache Hadoop locaux se composent d’un unique cluster volumineux qui prend en charge une grande quantité de charges de travail. Afin d’assurer un bon fonctionnement global, il se peut que ce cluster unique soit complexe et que des compromis soient nécessaires au niveau des services individuels. Pour migrer des clusters Hadoop locaux vers Azure HDInsight, vous devez adopter une nouvelle approche.

Les clusters Azure HDInsight sont conçus pour une utilisation de calcul de type spécifique. Étant donné qu’il est possible de partager le stockage entre plusieurs clusters, vous pouvez créer plusieurs clusters de calcul pour les charges de travail optimisées afin de répondre aux besoins des différents travaux. Chaque type de cluster est configuré de façon optimale pour une charge de travail spécifique. Le tableau suivant répertorie les types de clusters pris en charge dans HDInsight ainsi que les charges de travail correspondantes.

|Charge de travail|Type de cluster HDInsight|
|---|---|
|Traitement par lots (ETL/ELT)|Hadoop, Spark|
|Entrepôt de données|Hadoop, Spark, Interactive Query|
|IoT/diffusion en continu|Kafka, Storm, Spark|
|Traitement transactionnel NoSQL|hbase|
|Requêtes interactives et plus rapides avec la mise en cache en mémoire|Interactive Query|
|Science des données|ML Services, Spark|

Le tableau suivant présente les différentes méthodes permettant de créer un cluster HDInsight.

|Outil|Basé sur le navigateur|Ligne de commande|API REST|Kit SDK|
|---|---|---|---|---|
|[Azure portal](../hdinsight-hadoop-create-linux-clusters-portal.md)|X||||
|[Azure Data Factory](../hdinsight-hadoop-create-linux-clusters-adf.md).|X|X|X|X|
|[Azure CLI (version 1.0)](../hdinsight-hadoop-create-linux-clusters-azure-cli.md)||X|||
|[Azure PowerShell](../hdinsight-hadoop-create-linux-clusters-azure-powershell.md)||X|||
|[cURL](../hdinsight-hadoop-create-linux-clusters-curl-rest.md)||X|X||
|[Kit de développement logiciel (SDK) .NET](/dotnet/api/overview/azure/hdinsight?view=azure-dotnet&preserve-view=true)||||X|
|[Kit de développement logiciel (SDK) Python](/python/api/overview/azure/hdinsight)||||X|
|[Kit SDK Java](/java/api/overview/azure/hdinsight)||||X|
|[Modèles Microsoft Azure Resource Manager](../hdinsight-hadoop-create-linux-clusters-arm-templates.md)||X|||

Pour plus d’informations, consultez l’article [Types de clusters dans HDInsight](../hadoop/apache-hadoop-introduction.md).

## <a name="use-transient-on-demand-clusters"></a>Utiliser les clusters temporaires à la demande

Les clusters HDInsight peuvent être inutilisés pendant de longues périodes. Afin de réduire le coût des ressources, HDInsight prend en charge les clusters temporaires à la demande. Ces derniers peuvent être supprimés au terme de la charge de travail.

Lorsque vous supprimez un cluster, le compte de stockage associé et les métadonnées externes ne sont pas supprimés. Par la suite, le cluster peut être créé à nouveau à l’aide des mêmes comptes de stockage et metastores.

Vous pouvez utiliser Azure Data Factory pour planifier la création de clusters HDInsight à la demande. Pour plus d’informations, consultez l’article [Créer des clusters Apache Hadoop à la demande dans HDInsight avec Azure Data Factory](../hdinsight-hadoop-create-linux-clusters-adf.md).

## <a name="decouple-storage-from-compute"></a>Découpler le stockage du calcul

En général, les déploiements Hadoop locaux utilisent le même ensemble de machines pour le stockage des données et le traitement des données. Étant donné que ces deux opérations se trouvent au même emplacement, le calcul et le stockage doivent être mis à l’échelle conjointement.

Sur les clusters HDInsight, le stockage et le calcul peuvent être dissociés. Le stockage peut être effectué par le Stockage Azure, Azure Data Lake Storage, ou les deux. Le découplage du stockage et du calcul présente les avantages suivants :

- Partage de données entre clusters.
- Utilisation de clusters temporaires (car les données ne dépendent pas des clusters).
- Réduction des coûts de stockage.
- Mise à l’échelle séparée du stockage et du calcul.
- Réplication des données dans différentes régions.

Les clusters de calcul sont créés à proximité des ressources du compte de stockage dans une région Azure afin de réduire le coût de performance lié à la séparation du calcul et du stockage. Grâce aux réseaux à haut débit, les nœuds de calcul peuvent facilement accéder aux données au sein du stockage Azure.

## <a name="use-external-metadata-stores"></a>Utiliser des magasins de métadonnées externes

Deux principaux metastores fonctionnent avec les clusters HDInsight : [Apache Hive](https://hive.apache.org/) et [Apache Oozie](https://oozie.apache.org/). Le metastore Hive est le dépôt central de schémas qui peut être utilisé par les moteurs de traitement des données (y compris Hadoop, Spark, LLAP, Presto et Apache Pig). Le metastore Oozie stocke les détails à propos de la planification ainsi que l’état des travaux Hadoop en cours et achevés.

HDInsight utilise Azure SQL Database pour les metastores Hive et Oozie. Deux méthodes différentes permettent de configurer un metastore dans les clusters HDInsight :

1. Metastore par défaut

    - Pas de coût supplémentaire.
    - Le metastore supprimé en même temps que le cluster.
    - Le metastore ne peut pas être partagé entre différents clusters.
    - Utilisation de la base de données Azure SQL de base, qui est limitée à cinq DTU

1. Metastore externe personnalisé

    - Spécifiez une base de données Azure SQL externe comme metastore.
    - Les clusters peuvent être créés et supprimés sans perte des métadonnées (y compris le schéma Hive et les détails d’un travail Oozie).
    - Une seule base de données de metastore peut être partagée avec différents types de clusters.
    - Le metastore peut faire l’objet d’un scale-up selon les besoins.
    - Pour plus d’informations, consultez [Utiliser des magasins de métadonnées externes dans Azure HDInsight](../hdinsight-use-external-metadata-stores.md).

## <a name="best-practices-for-hive-metastore"></a>Meilleures pratiques pour les metastores Hive

Certaines des meilleures pratiques applicables aux metastores Hive dans HDInsight sont répertoriées ci-dessous :

- Utilisez un metastore externe personnalisé pour séparer les ressources de calcul des métadonnées.
- Commencez par une instance Azure SQL de niveau S2, qui fournit 50 DTU et 250 Go de stockage. Si vous voyez un goulot d’étranglement, vous pouvez mettre à l’échelle la base de données.
- Ne partagez pas le metastore créé pour une version de cluster HDInsight avec les clusters d’une autre version. Différentes versions d’Hive utilisent différents schémas. Par exemple, il est impossible de partager un metastore avec des clusters Hive 1.2 et 2.1.
- Sauvegardez régulièrement le metastore personnalisé.
- Conservez le metastore et le cluster HDInsight dans la même région.
- Surveillez les performances et la disponibilité du metastore à l'aide des outils de surveillance d'Azure SQL Database, comme le portail Azure ou les journaux Azure Monitor.
- Exécutez la commande `ANALYZE TABLE` comme indiqué afin de générer des statistiques pour les tables et les colonnes. Par exemple : `ANALYZE TABLE [table_name] COMPUTE STATISTICS`.

## <a name="best-practices-for-different-workloads"></a>Bonnes pratiques applicables aux différentes charges de travail

- Envisagez d’utiliser un cluster LLAP pour les requêtes Hive interactives avec des temps de réponse améliorés. [LLAP](https://cwiki.apache.org/confluence/display/Hive/LLAP) est une nouvelle fonctionnalité de Hive 2.0 qui permet la mise en cache en mémoire des requêtes. LLAP accélère considérablement les requêtes Hive, avec dans certains cas des vitesses jusqu’à [26 fois plus rapides qu’avec Hive 1.x](https://hortonworks.com/blog/announcing-apache-hive-2-1-25x-faster-queries-much/).
- Envisagez d’utiliser des travaux Spark à la place des travaux Hive.
- Envisagez de remplacer les requêtes basées sur les impala par des requêtes LLAP.
- Envisagez de remplacer des travaux MapReduce par des travaux Spark.
- Envisagez de remplacer les programmes de traitement par lots à faible latence Spark à l’aide de travaux Spark Structured Streaming.
- Envisagez d’utiliser Azure Data Factory (ADF) 2.0 pour l’orchestration des données.
- Envisagez d’utiliser Ambari pour la gestion des clusters.
- Remplacez le stockage de données HDFS local par WASB, ADLS ou ADFS pour le traitement des scripts.
- Envisagez d’utiliser Ranger RBAC sur les tables et l’audit Hive.
- Envisagez d’utiliser CosmosDB à la place de MongoDB ou Cassandra.

## <a name="next-steps"></a>Étapes suivantes

Lisez l’article suivant de cette série :

- [Bonnes pratiques concernant l’infrastructure pour une migration locale vers Azure HDInsight Hadoop](apache-hadoop-on-premises-migration-best-practices-infrastructure.md)