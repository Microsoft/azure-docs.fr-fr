---
title: Optimiser le stockage des données pour Apache Spark - Azure HDInsight
description: Découvrez comment optimiser le stockage des données pour une utilisation avec Apache Spark sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/20/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: ad09cdc2c0054c9d9a58e6bfa00252862f1e8c0f
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028221"
---
# <a name="data-storage-optimization-for-apache-spark"></a>Optimisation du stockage de données pour Apache Spark

Cet article décrit les stratégies permettant d’optimiser le stockage des données pour une exécution efficace des tâches Apache Spark sur Azure HDInsight.

## <a name="overview"></a>Vue d’ensemble

Spark prend en charge de nombreux formats, tels que csv, json, xml, parquet, orc et avro. Spark peut être étendu pour prendre en charge de nombreux autres formats avec des sources de données externes. Pour plus d’informations, consultez [Packages Apache Spark](https://spark-packages.org).

Le meilleur format du point de vue des performances est parquet avec *compression Snappy*, qui est l’option par défaut dans Spark 2.x. Parquet stocke les données sous forme de colonnes et il est fortement optimisé dans Spark.

## <a name="choose-data-abstraction"></a>Choisir l’abstraction des données

Les versions antérieures de Spark utilisent des RDD pour abstraire des données ; Spark 1.3 et Spark 1.6 ont introduit respectivement les DataFrames et les DataSets. Voici leurs avantages relatifs respectifs :

* **DataFrames**
    * Meilleur choix dans la plupart des cas
    * Fournit l’optimisation des requêtes par le biais de Catalyst
    * Génération de code à l’échelle globale
    * Accès direct à la mémoire
    * Faible surcharge de garbage collection
    * Convivialité inférieure à celles des DataSets pour les développeurs, car il n’y a aucune programmation d’objet de domaine ni vérification au moment de la compilation
* **DataSets**
    * Convient aux pipelines ETL complexes où l’impact sur les performances est acceptable
    * Ne convient pas dans les agrégations où l’impact sur les performances peut être considérable
    * Fournit l’optimisation des requêtes par le biais de Catalyst
    * Convivialité pour les développeurs, grâce à la programmation des objets de domaine et à la vérification au moment de la compilation
    * Ajoute une surcharge de sérialisation/désérialisation
    * Surcharge de garbage collection élevée
    * Interrompt la génération de code à l’échelle globale
* **RDD**
    * Vous n’avez pas besoin d’utiliser des RDD, sauf si vous devez générer un nouvel RDD personnalisé.
    * Aucune optimisation de requête par le biais de Catalyst
    * Aucune génération de code à l’échelle globale
    * Surcharge de garbage collection élevée
    * Nécessité d’utiliser des API Spark 1.x héritées

## <a name="select-default-storage"></a>Sélectionner le stockage par défaut

Quand vous créez un cluster Spark, vous pouvez sélectionner Stockage Blob Azure ou Azure Data Lake Storage comme stockage par défaut de votre cluster. Les deux options vous permettent de bénéficier d’un stockage à long terme pour les clusters transitoires. Ainsi, vos données ne sont pas supprimées automatiquement lorsque vous supprimez votre cluster. Vous pouvez recréer un cluster temporaire et encore accéder à vos données.

| Type de magasin | Système de fichiers | Vitesse | Temporaire | Cas d'utilisation |
| --- | --- | --- | --- | --- |
| Stockage Blob Azure | **wasb:** //url/ | **Standard** | Oui | Cluster temporaire |
| Stockage Blob Azure (sécurisé) | **wasbs:** //url/ | **Standard** | Oui | Cluster temporaire |
| Azure Data Lake Storage Gen 2| **abfs:** //url/ | **Plus rapide** | Oui | Cluster temporaire |
| Azure Data Lake Storage Gen 1| **adl:** //url/ | **Plus rapide** | Oui | Cluster temporaire |
| HDFS local | **hdfs:** //url/ | **Le plus rapide** | Non | Cluster 24/7 interactif |

Pour obtenir une description complète des options de stockage, consultez [Comparer les options de stockage à utiliser avec les clusters Azure HDInsight](../hdinsight-hadoop-compare-storage-options.md).

## <a name="use-the-cache"></a>Utiliser le cache

Spark fournit ses propres mécanismes de mise en cache native, que vous pouvez utiliser par le biais de différentes méthodes telles que `.persist()`, `.cache()` et `CACHE TABLE`. Cette mise en cache native est efficace avec les petits jeux de données et dans les pipelines ETL où vous devez mettre en cache des résultats intermédiaires. Toutefois, la mise en cache native de Spark ne fonctionne pas correctement avec le partitionnement, car une table mise en cache ne conserve pas les données de partitionnement. Une technique de mise en cache plus générique et plus fiable est *la mise en cache de la couche de stockage*.

* Mise en cache native de Spark (non recommandée)
    * Convient aux petits jeux de données
    * Ne fonctionne pas avec le partitionnement, mais ceci pourra changer dans les versions ultérieures de Spark

* Mise en cache au niveau du stockage (recommandée)
    * Peut être implémenté sur HDInsight à l’aide de la fonctionnalité [Cache d’E/S](apache-spark-improve-performance-iocache.md).
    * Utilise la mise en cache en mémoire et SSD

* HDFS local (recommandé)
    * Chemin `hdfs://mycluster`
    * Utilise la mise en cache SSD
    * Les données mises en cache sont perdues quand vous supprimez le cluster, ce qui nécessite une reconstruction du cache

## <a name="optimize-data-serialization"></a>Optimiser la sérialisation des données

Les travaux Spark étant distribués, une sérialisation des données appropriée est importante afin d’optimiser les performances.  Il existe deux options de sérialisation pour Spark :

* La sérialisation Java est l’option par défaut.
* La sérialisation `Kryo` est un format plus récent susceptible d’entraîner une sérialisation plus rapide et plus compacte que Java.  `Kryo` requiert l’inscription des classes dans votre programme et ne prend pas encore en charge tous les types sérialisables.

## <a name="use-bucketing"></a>Utiliser la création de compartiments

La création de compartiments est semblable au partitionnement des données, mais chaque compartiment peut contenir un ensemble de valeurs de colonne plutôt qu’une seule valeur. Cette méthode fonctionne bien pour le partitionnement sur un grand nombre de valeurs (plusieurs millions, voire plus), telles que les identificateurs de produits. Un compartiment est déterminé en hachant la clé de compartiment de la ligne. Les tables compartimentées offrent des optimisations uniques, car elles stockent des métadonnées relatives à la façon dont elles ont été compartimentées et triées.

Voici quelques fonctionnalités avancées offertes par les compartiments :

* Optimisation des requêtes en fonction des méta-informations de compartiments
* Optimisation des agrégations
* Optimisation des jointures

Vous pouvez utiliser le partitionnement et la création de compartiments en même temps.

## <a name="next-steps"></a>Étapes suivantes

* [Optimiser le traitement de données pour Apache Spark](optimize-cluster-configuration.md)
* [Optimiser l’utilisation de la mémoire pour Apache Spark](optimize-memory-usage.md)
* [Optimiser la configuration de cluster pour Apache Spark](optimize-cluster-configuration.md)
