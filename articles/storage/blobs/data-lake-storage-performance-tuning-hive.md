---
title: 'Régler les performances : Hive, HDInsight et Azure Data Lake Storage Gen2 | Microsoft Docs'
description: Comprendre les instructions de réglage pour les requêtes gourmandes en E/S à l’aide de Hive, HDInsight et Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2019
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: 4b1e5dd3c72122ade2fd4d4092bb18a7acf215f5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912941"
---
# <a name="tune-performance-hive-hdinsight--azure-data-lake-storage-gen2"></a>Régler les performances : Hive, HDInsight et Azure Data Lake Storage Gen2

Les paramètres par défaut ont été définis de manière à offrir de bonnes performances dans de nombreux cas d’usage différents.  Pour les requêtes intensives en E/S, vous pouvez régler Hive de manière à obtenir de meilleures performances avec Azure Data Lake Storage Gen2.  

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Data Lake Storage Gen2**. Pour obtenir les instructions de création de compte, consultez [Démarrage rapide : Créer un compte de stockage Azure Data Lake Storage Gen2](../common/storage-account-create.md)
* Un **cluster Azure HDInsight** avec un accès à un compte Data Lake Storage Gen2. Consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md)
* **Exécution de Hive sur HDInsight**.  Pour en savoir plus sur l’exécution de travaux Hive sur HDInsight, voir [Utiliser Hive dans HDInsight](../../hdinsight/hadoop/hdinsight-use-hive.md)
* **Conseils de réglage des performances sur Data Lake Storage Gen2**.  Pour en savoir plus sur les concepts de performance d’ordre général, consultez [Conseils de réglage des performances de Data Lake Storage Gen2](data-lake-storage-performance-tuning-guidance.md)

## <a name="parameters"></a>Paramètres

Voici les principaux paramètres à définir pour améliorer les performances de Data Lake Storage Gen2 :

* **hive.tez.container.size** : quantité de mémoire utilisée par chaque tâche

* **tez.grouping.min-size** : taille minimale de chaque mappeur

* **tez.grouping.max-size** : taille maximale de chaque mappeur

* **hive.exec.reducer.bytes.per.reducer** : taille de chaque réducteur

**hive.tez.container.size** : la taille du conteneur détermine la quantité de mémoire disponible pour chaque tâche.  Il s’agit de l’entrée principale permettant de contrôler l’accès concurrentiel dans Hive.  

**tez.grouping.min-size** : ce paramètre vous permet de définir la taille minimale de chaque mappeur.  Si le nombre de mappeurs choisis par Tez est inférieur à la valeur de ce paramètre, Tez utilise la valeur définie ici.

**tez.grouping.max-size** : ce paramètre vous permet de définir la taille maximale de chaque mappeur.  Si le nombre de mappeurs choisis par Tez est supérieur à la valeur de ce paramètre, Tez utilise la valeur définie ici.

**hive.exec.reducer.bytes.per.reducer** : ce paramètre définit la taille de chaque réducteur.  Par défaut, chaque réducteur a une taille de 256 Mo.  

## <a name="guidance"></a>Assistance

**Set hive.exec.reducer.bytes.per.reducer** : la valeur par défaut convient parfaitement lorsque les données sont décompressées.  Pour les données compressées, vous devez réduire la taille du réducteur.  

**Set hive.tez.container.size** : dans chaque nœud, la mémoire est spécifiée par yarn.nodemanager.resource.memory-mb et doit être correctement définie sur le cluster HDI par défaut.  Pour plus d’informations sur la configuration de mémoire appropriée dans YARN, consultez ce [billet](../../hdinsight/hdinsight-hadoop-hive-out-of-memory-error-oom.md).

Des charges de travail d’e/s intensives peuvent bénéficier de davantage de parallélisme grâce à la diminution de la taille du conteneur Tez. Cela fournit à l’utilisateur un nombre supérieur de conteneurs, ce qui augmente l’accès concurrentiel.  Toutefois, certaines requêtes Hive nécessitent une quantité importante de mémoire (par exemple, MapJoin).  Si la tâche ne dispose pas de suffisamment de mémoire, une exception de mémoire insuffisante se produit lors de l’exécution.  En cas d’exceptions de mémoire insuffisante, vous devez augmenter la mémoire.   

Le nombre de tâches simultanées en cours d’exécution ou le parallélisme sera limité par la mémoire YARN totale.  Le nombre de conteneurs YARN détermine le nombre de tâches simultanées pouvant être exécutées.  Pour rechercher la mémoire YARN par nœud, vous pouvez accéder à Ambari.  Accédez à YARN et affichez l’onglet Configurations.  La mémoire YARN s’affiche dans cette fenêtre.  

- Mémoire YARN totale = nœuds * mémoire YARN par nœud
- \# de conteneurs YARN = mémoire YARN totale/taille de conteneur Tez

La clé de l’amélioration des performances à l’aide de Data Lake Storage Gen2 consiste à augmenter la concurrence autant que possible.  Tez calcule automatiquement le nombre de tâches à créer pour que vous ne deviez pas le définir.   

## <a name="example-calculation"></a>Exemple de calcul

Supposons que vous disposiez d’un cluster D14 à 8 nœuds.  

- Mémoire YARN totale = nœuds * mémoire YARN par nœud
- Mémoire YARN totale = 8 nœuds * 96 Go = 768 Go
- \# de conteneurs YARN = 768 Go/3 072 Mo = 256

## <a name="further-information-on-hive-tuning"></a>Informations supplémentaires sur l’optimisation de Hive

Voici quelques blogs qui vous aideront à optimiser vos requêtes Hive :
* [Optimisation des requêtes Hive pour Hadoop dans HDInsight](../../hdinsight/hdinsight-hadoop-optimize-hive-query.md)
* [Optimiser les requêtes Apache Hive dans Azure HDInsight](../../hdinsight/hdinsight-hadoop-optimize-hive-query.md)
* [Lancement de la discussion sur l’optimisation de Hive sur HDInsight](https://channel9.msdn.com/events/Machine-Learning-and-Data-Sciences-Conference/Data-Science-Summit-2016/MSDSS25)