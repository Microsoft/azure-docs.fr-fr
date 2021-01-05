---
title: Optimisation du niveau de performance – Hive sur Azure Data Lake Storage Gen1
description: Découvrez l’optimisation des performances pour Hive sur HDInsight et Azure Data Lake Storage Gen1. Pour les requêtes intensives d’E/S, paramétrez Hive de manière à obtenir de meilleures performances.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 12/19/2016
ms.author: twooley
ms.openlocfilehash: 082d684ed0a29cb6bf2de9c506886b6d98cf174a
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97723779"
---
# <a name="performance-tuning-guidance-for-hive-on-hdinsight-and-azure-data-lake-storage-gen1"></a>Recommandations en matière d’optimisation des performances pour Hive sur HDInsight et Azure Data Lake Storage Gen1

Les paramètres par défaut ont été définis de manière à offrir de bonnes performances dans de nombreux cas d’usage différents.  Pour les requêtes intensives d’e/s, Hive peut être paramétré de manière à obtenir de meilleures performances avec Azure Data Lake Storage Gen1.  

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Data Lake Storage Gen1**. Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).
* **Cluster Azure HDInsight** avec accès à un compte Data Lake Storage Gen1. Voir [Créer un cluster HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md). Veillez à activer le Bureau à distance pour le cluster.
* **Exécution de Hive sur HDInsight**.  Pour en savoir plus sur l’exécution de travaux Hive sur HDInsight, voir [Utiliser Hive dans HDInsight](../hdinsight/hadoop/hdinsight-use-hive.md)
* **Recommandations en matière d’optimisation des performances sur Data Lake Storage Gen1**.  Pour des concepts généraux sur les performances, consultez [Recommandations en matière d’optimisation des performances de Data Lake Storage Gen1](./data-lake-store-performance-tuning-guidance.md)

## <a name="parameters"></a>Paramètres

Voici les principaux paramètres à définir pour améliorer les performances de Data Lake Storage Gen1 :

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

**Set hive.tez.container.size** : dans chaque nœud, la mémoire est spécifiée par yarn.nodemanager.resource.memory-mb et doit être correctement définie sur le cluster HDI par défaut.  Pour plus d’informations sur la configuration de mémoire appropriée dans YARN, consultez ce [billet](../hdinsight/hdinsight-hadoop-hive-out-of-memory-error-oom.md).

Des charges de travail d’e/s intensives peuvent bénéficier de davantage de parallélisme grâce à la diminution de la taille du conteneur Tez. Cela fournit à l’utilisateur un nombre supérieur de conteneurs, ce qui augmente l’accès concurrentiel.  Toutefois, certaines requêtes Hive nécessitent une quantité importante de mémoire (par exemple, MapJoin).  Si la tâche ne dispose pas de suffisamment de mémoire, une exception de mémoire insuffisante se produit lors de l’exécution.  En cas d’exceptions de mémoire insuffisante, vous devez augmenter la mémoire.   

Le nombre de tâches simultanées en cours d’exécution ou le parallélisme sera limité par la mémoire YARN totale.  Le nombre de conteneurs YARN détermine le nombre de tâches simultanées pouvant être exécutées.  Pour rechercher la mémoire YARN par nœud, vous pouvez accéder à Ambari.  Accédez à YARN et affichez l’onglet Configurations.  La mémoire YARN s’affiche dans cette fenêtre.  

> Mémoire YARN totale = nœuds * mémoire YARN par nœud Nombre de conteneurs YARN = total de la mémoire YARN / taille du conteneur Tez

La clé de l’amélioration des performances à l’aide de Data Lake Storage Gen1 consiste à augmenter la concurrence autant que possible.  Tez calcule automatiquement le nombre de tâches à créer pour que vous ne deviez pas le définir.   

## <a name="example-calculation"></a>Exemple de calcul

Supposons que vous disposiez d’un cluster D14 à 8 nœuds.  

> Mémoire YARN totale = nœuds * mémoire YARN par nœud Mémoire YARN totale = 8 nœuds * 96 Go = 768 GB Nombre de conteneurs YARN = 768 GB / 3072 MB = 256

## <a name="limitations"></a>Limites

**Limitation de bande passante Data Lake Storage Gen1** 

Vous pouvez constater des échecs de tâche si vous atteignez les limites de la bande passante fournie par Data Lake Storage Gen1. Vous pouvez identifier le problème en consultant les erreurs de limitation dans les journaux d’activité des tâches.  Vous pouvez réduire le parallélisme en augmentant la taille du conteneur Tez.  Si vous avez besoin de davantage de simultanéité pour votre travail, veuillez nous contacter.

Pour vérifier si une limitation est appliquée, vous devez activer la journalisation du débogage côté client. Voici comment procéder :

1. Placez la propriété suivante dans les propriétés log4j Hive config. Cela peut se faire à partir de la vue d’Ambari : log4j.logger.com.microsoft.azure.datalake.store=DEBUG Redémarrez tous les nœuds/le service pour que la configuration soit appliquée.

2. Si vous êtes limité, le code d’erreur HTTP 429 s’affiche dans le fichier journal Hive. Le fichier journal Hive se trouve dans /tmp/&lt;utilisateur&gt;/hive.log

## <a name="further-information-on-hive-tuning"></a>Informations supplémentaires sur l’optimisation de Hive

Voici quelques blogs qui vous aideront à optimiser vos requêtes Hive :
* [Optimisation des requêtes Hive pour Hadoop dans HDInsight](../hdinsight/hdinsight-hadoop-optimize-hive-query.md)
* [Encodage du fichier de requête Hive dans Azure HDInsight](/archive/blogs/bigdatasupport/encoding-the-hive-query-file-in-azure-hdinsight)
* [Lancement de la discussion sur l’optimisation de Hive sur HDInsight](https://channel9.msdn.com/events/Machine-Learning-and-Data-Sciences-Conference/Data-Science-Summit-2016/MSDSS25)