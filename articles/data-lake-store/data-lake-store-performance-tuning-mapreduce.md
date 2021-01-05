---
title: Optimisation des performances d’Azure Data Lake Storage Gen1 - MapReduce
description: Découvrez l’optimisation des performances pour MapReduce dans Azure Data Lake Storage Gen1, notamment les paramètres, l’aide, un exemple de calcul et les limitations.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 12/19/2016
ms.author: twooley
ms.openlocfilehash: 2549413241e422fb1e0e5e1f079c287e0b7cf005
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97723726"
---
# <a name="performance-tuning-guidance-for-mapreduce-on-hdinsight-and-azure-data-lake-storage-gen1"></a>Recommandations en matière d’optimisation des performances pour MapReduce sur HDInsight et Azure Data Lake Storage Gen1

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Data Lake Storage Gen1**. Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).
* **Cluster Azure HDInsight** avec accès à un compte Data Lake Storage Gen1. Voir [Créer un cluster HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md). Veillez à activer le Bureau à distance pour le cluster.
* **Utilisation de MapReduce sur HDInsight**. Pour plus d’informations, consultez [Utilisation de MapReduce sur Hadoop sur HDInsight](../hdinsight/hadoop/hdinsight-use-mapreduce.md)
* **Passez en revue les recommandations en matière d’optimisation des performances pour Data Lake Storage Gen1**. Pour des concepts généraux sur les performances, consultez [Recommandations en matière d’optimisation des performances de Data Lake Storage Gen1](./data-lake-store-performance-tuning-guidance.md)

## <a name="parameters"></a>Paramètres

Lors de l’exécution des travaux MapReduce, voici les paramètres les plus importants que vous pouvez configurer pour améliorer les performances sur Data Lake Storage Gen1 :

|Paramètre      | Description  |
|---------|---------|
|`Mapreduce.map.memory.mb`  |  Quantité de mémoire à allouer à chaque mappeur.  |
|`Mapreduce.job.maps`     |  Nombre de tâches de mappage par travail.  |
|`Mapreduce.reduce.memory.mb`     |  Quantité de mémoire à allouer à chaque réducteur.  |
|`Mapreduce.job.reduces`    |   Nombre de tâches de réduction par travail.  |

### <a name="mapreducemapmemory--mapreducereducememory"></a>Mapreduce.map.memory / Mapreduce.reduce.memory

Ajustez ce nombre en fonction de la quantité de mémoire nécessaire pour la tâche de mappage et/ou de réduction. Vous pouvez afficher les valeurs par défaut de `mapreduce.map.memory` et de `mapreduce.reduce.memory` dans Ambari à l’aide de la configuration Yarn. Dans Ambari, accédez à YARN et affichez l’onglet **Configurations**. La mémoire YARN s’affiche.

### <a name="mapreducejobmaps--mapreducejobreduces"></a>Mapreduce.job.maps / Mapreduce.job.reduces

Détermine le nombre maximal de mappeurs ou de réducteurs à créer. Le nombre de fractionnements détermine le nombre de mappeurs créés pour le travail MapReduce. Ainsi, vous pouvez obtenir moins de mappeurs que demandé s’il existe moins de fractionnements que de mappeurs demandés.

## <a name="guidance"></a>Assistance

### <a name="step-1-determine-number-of-jobs-running"></a>Étape 1 : Déterminer le nombre de travaux en cours d’exécution

Par défaut, MapReduce utilise l’ensemble du cluster pour votre travail. Vous pouvez utiliser une partie moindre du cluster en utilisant moins de mappeurs que de conteneurs disponibles. Les instructions de ce document supposent que votre application est la seule application en cours d’exécution sur votre cluster.

### <a name="step-2-set-mapreducemapmemorymapreducereducememory"></a>Étape 2 : Configurer mapreduce.map.memory/mapreduce.reduce.memory

La taille de la mémoire pour les tâches de mappage et de réduction dépend du travail en question. Vous pouvez réduire la taille de la mémoire si vous souhaitez augmenter la simultanéité. Le nombre de tâches exécutées simultanément varie selon le nombre de conteneurs. En réduisant la quantité de mémoire par mappeur ou réducteur, il est possible de créer plus de conteneurs, pour permettre à plus de mappeurs ou réducteurs de s’exécuter simultanément. Trop réduire la quantité de mémoire risque de causer des problèmes de mémoire insuffisante pour certains processus. Si vous obtenez une erreur de segment de mémoire lors de l’exécution de votre travail, augmentez la mémoire par mappeur ou réducteur. Prenez en compte le fait que l’ajout de conteneurs ajoute une charge pour chaque conteneur supplémentaire, ce qui peut dégrader les performances. Une autre solution consiste à obtenir davantage de mémoire à l’aide d’un cluster qui possède une quantité supérieure de mémoire ou en augmentant le nombre de nœuds de votre cluster. Une mémoire plus importante permettra d’utiliser plus de conteneurs, pour plus de simultanéité.

### <a name="step-3-determine-total-yarn-memory"></a>Étape 3 : Déterminer la mémoire YARN totale

Pour régler mapreduce.job.maps/mapreduce.job.reduces, prenez en compte la quantité de mémoire YARN totale disponible à utiliser. Ces informations sont disponibles dans Ambari. Accédez à YARN et affichez l’onglet **Configurations**. La mémoire YARN s’affiche dans cette fenêtre. Multipliez la mémoire YARN par le nombre de nœuds dans votre cluster pour obtenir la mémoire YARN totale.

`Total YARN memory = nodes * YARN memory per node`

Si vous utilisez un cluster vide, la mémoire peut être la mémoire YARN totale pour votre cluster. Si d’autres applications utilisent de la mémoire, vous pouvez choisir d’utiliser uniquement une partie de la mémoire de votre cluster en réduisant le nombre de mappeurs ou réducteurs au nombre de conteneurs que vous souhaitez utiliser.

### <a name="step-4-calculate-number-of-yarn-containers"></a>Étape 4 : Calculer le nombre de conteneurs YARN

Les conteneurs YARN déterminent la quantité de simultanéité disponible pour le travail. Prenez la mémoire YARN totale et divisez-la par mapreduce.map.memory.

`# of YARN containers = total YARN memory / mapreduce.map.memory`

### <a name="step-5-set-mapreducejobmapsmapreducejobreduces"></a>Étape 5 : Configurer mapreduce.job.maps/mapreduce.job.reduces

Affectez à mapreduce.job.maps/mapreduce.job.reduces une valeur au moins égale au nombre de conteneurs disponibles. Vous pouvez expérimenter davantage en augmentant le nombre de mappeurs et de réducteurs pour voir si vous obtenez de meilleures performances. N’oubliez pas que les mappeurs supplémentaires ajouteront une charge, ainsi un trop grand nombre de mappeurs peut dégrader les performances.

La planification et l’isolation de processeur sont désactivées par défaut. Par conséquent, le nombre de conteneurs YARN est donc limité par la mémoire.

## <a name="example-calculation"></a>Exemple de calcul

Supposons que vous avez actuellement un cluster composé de 8 nœuds D14 et que vous souhaitez exécuter une tâche intensive en E/S. Voici les calculs que vous devez faire :

### <a name="step-1-determine-number-of-jobs-running"></a>Étape 1 : Déterminer le nombre de travaux en cours d’exécution

Dans notre exemple, nous supposons que notre travail est le seul en cours d’exécution.

### <a name="step-2-set-mapreducemapmemorymapreducereducememory"></a>Étape 2 : Configurer mapreduce.map.memory/mapreduce.reduce.memory

Pour notre exemple, vous exécutez une tâche intensive en E/S et nous décidons que 3 Go de mémoire pour les tâches de mappage suffisent.

`mapreduce.map.memory = 3GB`

### <a name="step-3-determine-total-yarn-memory"></a>Étape 3 : Déterminer la mémoire YARN totale

`total memory from the cluster is 8 nodes * 96GB of YARN memory for a D14 = 768GB`

### <a name="step-4-calculate--of-yarn-containers"></a>Étape 4 : Calculer le nombre de conteneurs YARN

`# of YARN containers = 768 GB of available memory / 3 GB of memory = 256`

### <a name="step-5-set-mapreducejobmapsmapreducejobreduces"></a>Étape 5 : Configurer mapreduce.job.maps/mapreduce.job.reduces

`mapreduce.map.jobs = 256`

## <a name="limitations"></a>Limites

**Limitation de bande passante Data Lake Storage Gen1**

En tant que service mutualisé, Data Lake Storage Gen1 définit des limites de bande passante de niveau de compte. Si vous rencontrez ces limites, vous commencerez à observer des échecs pour vos tâches. Vous pouvez identifier le problème en consultant les erreurs de limitation dans les journaux d’activité des tâches. Si vous avez besoin de davantage de bande passante pour votre travail, veuillez nous contacter.

Pour vérifier si une limitation est appliquée, vous devez activer la journalisation du débogage côté client. Voici comment procéder :

1. Placez la propriété suivante dans les propriétés de log4j dans Ambari > YARN > Config > Avancé yarn-log4j: log4j.logger.com.microsoft.azure.datalake.store=DEBUG

2. Redémarrez tous les nœuds/le service pour que la configuration prenne effet.

3. Si vous êtes limité, vous verrez le code d’erreur HTTP 429 dans le fichier journal YARN. Le fichier journal YARN se trouve dans /tmp/&lt;utilisateur&gt;/yarn.log

## <a name="examples-to-run"></a>Exemples à exécuter

Pour illustrer comment MapReduce s’exécute sur Data Lake Storage Gen1, voici un exemple de code qui a été exécuté sur un cluster avec les paramètres suivants :

* D14v2 16 nœuds
* Cluster Hadoop exécutant HDI 3.6

Pour commencer, voici quelques exemples de commandes pour exécuter MapReduce Teragen, Terasort et Teravalidate. Vous pouvez ajuster ces commandes en fonction de vos ressources.

### <a name="teragen"></a>Teragen

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 10000000000 adl://example/data/1TB-sort-input
```

### <a name="terasort"></a>Terasort

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 -Dmapreduce.job.reduces=512 -Dmapreduce.reduce.memory.mb=3072 adl://example/data/1TB-sort-input adl://example/data/1TB-sort-output
```

### <a name="teravalidate"></a>Teravalidate

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapreduce.job.maps=512 -Dmapreduce.map.memory.mb=3072 adl://example/data/1TB-sort-output adl://example/data/1TB-sort-validate
```