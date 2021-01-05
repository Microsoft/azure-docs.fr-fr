---
title: Optimisation des performances – Spark avec Azure Data Lake Storage Gen1
description: Découvrez les conseils d’optimisation des performances pour Spark sur Azure HDInsight et Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 12/19/2016
ms.author: twooley
ms.openlocfilehash: 8a39e14b70827947687b7613b9ff86f18017ddfe
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97722502"
---
# <a name="performance-tuning-guidance-for-spark-on-hdinsight-and-azure-data-lake-storage-gen1"></a>Recommandations en matière d’optimisation des performances pour Spark sur HDInsight et Azure Data Lake Storage Gen1

Lors du réglage des performances sur Spark, vous devez prendre en compte le nombre d’applications qui s’exécuteront sur votre cluster. Par défaut, vous pouvez exécuter quatre applications simultanément sur votre cluster HDI (remarque : le paramétrage par défaut est susceptible de changer). Vous pouvez décider d’utiliser moins d’applications, vous pouvez donc remplacer les paramètres par défaut et utiliser le cluster davantage pour ces applications.

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Data Lake Storage Gen1**. Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).
* **Cluster Azure HDInsight** avec accès à un compte Data Lake Storage Gen1. Voir [Créer un cluster HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md). Veillez à activer le Bureau à distance pour le cluster.
* **Exécution d’un cluster Spark sur Data Lake Storage Gen1**. Pour plus d’informations, consultez [Utiliser le cluster HDInsight Spark pour analyser les données dans Data Lake Storage Gen1](../hdinsight/spark/apache-spark-use-with-data-lake-store.md)
* **Recommandations en matière d’optimisation des performances sur Data Lake Storage Gen1**. Pour des concepts généraux sur les performances, consultez [Recommandations en matière d’optimisation des performances de Data Lake Storage Gen1](./data-lake-store-performance-tuning-guidance.md) 

## <a name="parameters"></a>Paramètres

Lors de l’exécution de travaux Spark, voici les principaux paramètres qui peuvent être paramétrés pour améliorer les performances sur Data Lake Storage Gen1 :

* **Num-executors** : le nombre de tâches simultanées qui peuvent être exécutées.

* **Executor-memory** : la quantité de mémoire allouée à chaque exécuteur.

* **Executor-cores** : le nombre de cœurs alloués à chaque exécuteur.

**Num-executors** Num-executors définit le nombre maximal de tâches pouvant s’exécuter en parallèle. Le nombre réel de tâches qui peuvent s’exécuter en parallèle est limité par la mémoire et les ressources processeur disponibles dans votre cluster.

**Executor-memory** : la quantité de mémoire allouée à chaque exécuteur. La mémoire nécessaire pour chaque exécuteur dépend de la tâche. Pour les opérations complexes, la mémoire doit être plus élevée. Pour les opérations simples telles que la lecture et l’écriture, les besoins en mémoire seront inférieurs. La quantité de mémoire pour chaque exécuteur peut être consultée dans Ambari. Dans Ambari, accédez à Spark et affichez l’onglet **Configurations**.

**Executor-cores** : définit la quantité de cœurs utilisés par exécuteur, ce qui détermine le nombre de threads parallèles qui peuvent être exécutés par l’exécuteur. Par exemple, si Executor-cores = 2, alors chaque exécuteur peut exécuter des 2 tâches parallèles dans l’exécuteur. La valeur à utiliser pour Executor-cores dépendra de la tâche. Les travaux intensifs en E/S ne nécessitent pas une grande quantité de mémoire par la tâche, ainsi chaque exécuteur peut gérer davantage de tâches parallèles.

Par défaut, deux cœurs YARN virtuels sont définis pour chaque cœur physique lors de l’exécution de Spark sur HDInsight. Ce nombre offre un bon équilibre de simultanéité et de quantité de changement de contexte à partir de plusieurs threads.

## <a name="guidance"></a>Assistance

Lorsque vous exécutez des charges de travail analytiques Spark pour l’utilisation de données dans Data Lake Storage Gen1, nous vous recommandons d’utiliser la version la plus récente de HDInsight pour bénéficier des meilleures performances avec Data Lake Storage Gen1. Lorsque votre travail nécessite une grande quantité d’E/S, vous pouvez configurer certains paramètres pour améliorer les performances. Data Lake Storage Gen1 est une plateforme de stockage hautement évolutive capable de gérer un débit élevé. Si le travail consiste principalement en une série de lectures ou d’écritures, l’augmentation de la concurrence pour les E/S à destination et en provenance de Data Lake Storage Gen1 peut accroître les performances.

Vous disposez de plusieurs méthodes générales pour augmenter la concurrence des travaux qui nécessitent une grande quantité d’E/S.

**Étape 1 : Déterminer le nombre d’applications s’exécutant sur votre cluster** : vous devez connaître le nombre d’applications en cours d’exécution sur le cluster, y compris celle en cours. Les valeurs par défaut pour chaque paramètre Spark supposent que 4 applications s’exécutent simultanément. Par conséquent, vous avez seulement 25 % du cluster disponible pour chaque application. Pour obtenir de meilleures performances, vous pouvez remplacer les valeurs par défaut en modifiant le nombre d’exécuteurs.

**Étape 2 : Configurer executor-memory** : la première chose à configurer est executor-memory. La mémoire sera dépendante de la tâche que vous exécutez. Vous pouvez augmenter la simultanéité en allouant de moins de mémoire par exécuteur. Si vous voyez des exceptions de mémoire insuffisante lors de l’exécution de votre tâche, vous devez augmenter la valeur de ce paramètre. Une solution consiste à obtenir davantage de mémoire à l’aide d’un cluster qui possède une quantité supérieure de mémoire ou en augmentant la taille de votre cluster. Une mémoire plus importante permettra d’utiliser plus d’exécuteurs, pour plus de simultanéité.

**Étape 3 : Définir executor-cores** : pour les charges de travail intensives qui n’ont pas d’opérations complexes, il est conseillé de commencer avec une valeur executor-cores élevée pour augmenter le nombre de tâches parallèles par exécuteur. Régler Executor-cores sur 4 est un bon point de départ.

```console
executor-cores = 4
```

Augmenter la valeur d’Executor-cores vous donne plus de parallélisme, vous pouvez donc expérimenter différentes valeurs pour ce paramètre. Pour les tâches qui ont des opérations plus complexes, vous devez réduire le nombre de cœurs par exécuteur. Si la valeur d’Executor-cores est supérieure à 4, puis le garbage collection peut devenir inefficace et dégrader les performances.

**Étape 4 : Déterminer la quantité de mémoire YARN du cluster** : cette information est disponible dans Ambari. Accédez à YARN et affichez l’onglet Configurations. La mémoire YARN s’affiche dans cette fenêtre.
Remarque : lorsque vous êtes dans la fenêtre, vous pouvez également voir la taille de conteneur YARN par défaut. La taille du conteneur YARN est identique au paramètre de mémoire par exécuteur.

Mémoire YARN totale = nœuds * mémoire YARN par nœud

**Étape 5 : Calculer num-executors**

**Calculer la contrainte de mémoire** : le paramètre num-executors est limité par la mémoire ou par le processeur. La contrainte de mémoire est déterminée par la quantité de mémoire YARN disponible pour votre application. Prenez la mémoire YARN totale et divisez-la par la valeur de executor-memory. La mise à l’échelle de la contrainte doit être adaptée pour le nombre d’applications, nous la divisons donc par le nombre d’applications.

Contrainte de mémoire = (total de la mémoire YARN / mémoire de l’exécuteur) / nombre d’applications

**Calculer la contrainte de processeur** : la contrainte de processeur est calculée comme le nombre total de cœurs virtuels divisé par le nombre de cœurs par exécuteur. Il existe 2 cœurs virtuels pour chaque noyau physique. Comme pour la contrainte de mémoire, nous divisons par le nombre d’applications.

cœurs virtuels = (nœuds dans le cluster * nombre de cœurs physiques dans nœud * 2) contrainte processeur = (nombre total de cœurs virtuels / nombre de cœurs par exécuteur) / nombre d’applications

**Définir num-executors** : le paramètre num-executors est déterminé par la valeur minimale entre la contrainte de mémoire et la contrainte de processeur. 

num-executors = min (nombre total de cœurs virtuels / nombre de cœurs par exécuteur, mémoire YARN disponible / mémoire exécuteur) Le fait de configurer un plus grand nombre de num-executors n’améliore pas nécessairement les performances. Vous devez prendre en compte le fait que l’ajout d’exécuteurs ajoute une charge pour chaque exécuteur supplémentaire, ce qui peut dégrader les performances. Num-executors est limité par les ressources de cluster.

## <a name="example-calculation"></a>Exemple de calcul

Supposons que vous possédez un cluster composé de 8 nœuds D4v2 exécutant 2 applications, dont celle que vous allez exécuter.

**Étape 1 : Déterminer le nombre d’applications s’exécutant sur votre cluster** : vous savez que vous avez deux applications sur votre cluster, dont celle que vous allez exécuter.

**Étape 2 : Configurer executor-memory** : pour cet exemple, nous déterminons que 6 Go pour executor-memory seront suffisants pour les travaux intensifs en E/S.

```console
executor-memory = 6GB
```

**Étape 3 : Définir executor-cores** : dans la mesure où il s’agit d’un travail intensif en E/S, nous pouvons définir le nombre de cœurs pour chaque exécuteur sur quatre. Définir le nombre de cœurs par exécuteur sur une valeur supérieure à quatre peut provoquer des problèmes de garbage collection.

```console
executor-cores = 4
```

**Étape 4 : Déterminer la quantité de mémoire YARN du cluster** : nous accédons à Ambari pour déterminer que chaque D4v2 possède 25 Go de mémoire YARN. Étant donné qu’il y a 8 nœuds, la mémoire YARN disponible est multipliée par 8.

Mémoire YARN totale = nœuds * mémoire YARN * par nœud Mémoire YARN totale = 8 nœuds * 25 Go = 200 Go

**Étape 5 : Calculer num-executors** : le paramètre num-executors est déterminé par la valeur minimale entre la contrainte de mémoire et la contrainte de processeur, divisée par le nombre d’applications exécutées sur Spark.

**Calculer la contrainte de mémoire** : la contrainte de mémoire est calculée comme la mémoire YARN totale divisée par la mémoire par exécuteur.

Contrainte mémoire = (mémoire YARN totale / mémoire de l’exécuteur) / nombre d’applications Contrainte mémoire = (200 Go / 6 Go) / 2 Contrainte mémoire = 16 (arrondi) **Calculer la contrainte processeur** - La contrainte processeur est calculée comme le nombre total de cœurs Yarn divisé par le nombre de cœurs par exécuteur.

Cœurs YARN = nœuds dans le cluster * nombre de cœurs par nœud * 2 Cœurs YARN = 8 nœuds * 8 cœurs par D14 * 2 = 128 Contrainte processeur = (nombre total de cœurs YARN / nombre de cœurs par exécuteur) / nombre d’applications Contrainte processeur = (128 / 4) / 2 Contrainte processeur = 16

**Définir num-executors**

num-executors = Min (contrainte mémoire, contrainte processeur) num-executors = Min (16, 16) num-executors = 16