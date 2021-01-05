---
title: 'Régler les performances : Spark, HDInsight et Azure Data Lake Storage Gen2 | Microsoft Docs'
description: Comprendre les recommandations relatives au réglage des performances Spark avec Azure HDInsight et Azure Data Lake Storage Gen2.
services: storage
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2019
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: c99d57ddd86ecff71c35ad6c0f2c2561e279b4b0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912805"
---
# <a name="tune-performance-spark-hdinsight--azure-data-lake-storage-gen2"></a>Régler les performances : Spark, HDInsight et Azure Data Lake Storage Gen2

Lors du réglage des performances sur Spark, vous devez prendre en compte le nombre d’applications qui s’exécuteront sur votre cluster.  Par défaut, vous pouvez exécuter 4 applications simultanément sur votre cluster HDI (Remarque : le paramètre par défaut est susceptible de changer).  Vous pouvez décider d’utiliser moins d’applications, vous pouvez donc remplacer les paramètres par défaut et utiliser le cluster davantage pour ces applications.  

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Data Lake Storage Gen2**. Pour obtenir les instructions de création de compte, consultez [Démarrage rapide : Créer un compte de stockage Azure Data Lake Storage Gen2](../common/storage-account-create.md).
* Un **cluster Azure HDInsight** avec un accès à un compte Data Lake Storage Gen2. Consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md). Veillez à activer le Bureau à distance pour le cluster.
* **Exécution d’un cluster Spark sur Data Lake Storage Gen2**.  Pour plus d’informations, consultez [Utiliser le cluster HDInsight Spark pour analyser les données dans Data Lake Storage Gen2](../../hdinsight/spark/apache-spark-use-with-data-lake-store.md)
* **Conseils de réglage des performances sur Data Lake Storage Gen2**.  Pour en savoir plus sur les concepts de performance d’ordre général, consultez [Conseils de réglage des performances de Data Lake Storage Gen2](data-lake-storage-performance-tuning-guidance.md) 

## <a name="parameters"></a>Paramètres

Lors de l’exécution de travaux Spark, voici les principaux paramètres qui peuvent être réglés pour améliorer les performances sur Data Lake Storage Gen2 :

* **Num-executors** : le nombre de tâches simultanées qui peuvent être exécutées.

* **Executor-memory** : la quantité de mémoire allouée à chaque exécuteur.

* **Executor-cores** : le nombre de cœurs alloués à chaque exécuteur.                     

**Num-executors** Num-executors définit le nombre maximal de tâches pouvant s’exécuter en parallèle.  Le nombre réel de tâches qui peuvent s’exécuter en parallèle est limité par la mémoire et les ressources processeur disponibles dans votre cluster.

**Executor-memory** : la quantité de mémoire allouée à chaque exécuteur.  La mémoire nécessaire pour chaque exécuteur dépend de la tâche.  Pour les opérations complexes, la mémoire doit être plus élevée.  Pour les opérations simples telles que la lecture et l’écriture, les besoins en mémoire seront inférieurs.  La quantité de mémoire pour chaque exécuteur peut être consultée dans Ambari.  Dans Ambari, accédez à Spark et affichez l’onglet Configurations.  

**Executor-cores** : définit le nombre de cœurs utilisés par exécuteur, ce qui détermine le nombre de threads parallèles qui peuvent être exécutés par exécuteur.  Par exemple, si Executor-cores = 2, alors chaque exécuteur peut exécuter des 2 tâches parallèles dans l’exécuteur.  La valeur à utiliser pour Executor-cores dépendra de la tâche.  Les travaux intensifs en E/S ne nécessitent pas une grande quantité de mémoire par la tâche, ainsi chaque exécuteur peut gérer davantage de tâches parallèles.

Par défaut, deux cœurs YARN virtuels sont définis pour chaque cœur physique lors de l’exécution de Spark sur HDInsight.  Ce nombre offre un bon équilibre de simultanéité et de quantité de changement de contexte à partir de plusieurs threads.  

## <a name="guidance"></a>Assistance

Lorsque vous exécutez des charges de travail analytiques Spark pour l’utilisation de données dans Data Lake Storage Gen2, nous vous recommandons d’utiliser la version la plus récente de HDInsight pour bénéficier des meilleures performances avec Data Lake Storage Gen2. Lorsque votre travail nécessite une grande quantité d’E/S, vous pouvez configurer certains paramètres pour améliorer les performances.  Data Lake Storage Gen2 est une plateforme de stockage hautement évolutive capable de gérer un débit élevé.  Si le travail consiste principalement en une série de lectures ou d’écritures, l’augmentation de la concurrence pour les E/S à destination et en provenance de Data Lake Storage Gen2 peut accroître les performances.

Vous disposez de plusieurs méthodes générales pour augmenter la concurrence des travaux qui nécessitent une grande quantité d’E/S.

**Étape 1 : Déterminer le nombre d’applications s’exécutant sur votre cluster** : vous devez connaître le nombre d’applications en cours d’exécution sur le cluster, y compris celle en cours.  Les valeurs par défaut pour chaque paramètre Spark supposent que 4 applications s’exécutent simultanément.  Par conséquent, vous avez seulement 25 % du cluster disponible pour chaque application.  Pour obtenir de meilleures performances, vous pouvez remplacer les valeurs par défaut en modifiant le nombre d’exécuteurs.  

**Étape 2 : Configurer executor-memory** : la première chose à définir est executor-memory.  La mémoire sera dépendante de la tâche que vous exécutez.  Vous pouvez augmenter la simultanéité en allouant de moins de mémoire par exécuteur.  Si vous voyez des exceptions de mémoire insuffisante lors de l’exécution de votre tâche, vous devez augmenter la valeur de ce paramètre.  Une solution consiste à obtenir davantage de mémoire à l’aide d’un cluster qui possède une quantité supérieure de mémoire ou en augmentant la taille de votre cluster.  Une mémoire plus importante permettra d’utiliser plus d’exécuteurs, pour plus de simultanéité.

**Étape 3 : Définir executor-cores** : pour les charges de travail intensives qui n’ont pas d’opérations complexes, il est conseillé de commencer avec une valeur executor-cores élevée pour augmenter le nombre de tâches parallèles par exécuteur.  Régler Executor-cores sur 4 est un bon point de départ.   

executor-cores = 4

Augmenter la valeur d’Executor-cores vous donne plus de parallélisme, vous pouvez donc expérimenter différentes valeurs pour ce paramètre.  Pour les tâches qui ont des opérations plus complexes, vous devez réduire le nombre de cœurs par exécuteur.  Si la valeur d’Executor-cores est supérieure à 4, puis le garbage collection peut devenir inefficace et dégrader les performances.

**Étape 4 : Déterminer la quantité de mémoire YARN du cluster** : cette information est disponible dans Ambari.  Accédez à YARN et affichez l’onglet Configurations.  La mémoire YARN s’affiche dans cette fenêtre.  
Remarque : lorsque vous êtes dans la fenêtre, vous pouvez également voir la taille de conteneur YARN par défaut.  La taille du conteneur YARN est identique au paramètre de mémoire par exécuteur.

Mémoire YARN totale = nœuds * mémoire YARN par nœud

**Étape 5 : Calculer num-executors**

**Calculer la contrainte de mémoire** : le paramètre num-executors est limité par la mémoire ou par le processeur.  La contrainte de mémoire est déterminée par la quantité de mémoire YARN disponible pour votre application.  Vous devez prendre la mémoire YARN totale et la diviser par executor-memory.  La mise à l’échelle de la contrainte doit être adaptée pour le nombre d’applications, nous la divisons donc par le nombre d’applications.

Contrainte de mémoire = (total de la mémoire YARN / mémoire de l’exécuteur) / nombre d’applications

**Calculer la contrainte de processeur** : la contrainte de processeur est calculée comme le nombre total de cœurs virtuels divisé par le nombre de cœurs par exécuteur.  Il existe 2 cœurs virtuels pour chaque noyau physique.  Comme pour la contrainte de mémoire, nous divisons par le nombre d’applications.

- cœurs virtuels = (nœuds dans le cluster * nombre de cœurs physiques dans le nœud * 2)
- Contrainte UC = (nombre total de cœurs virtuels/nombre de cœurs par exécuteur)/nombre d’applications

**Définir num-executors** : le paramètre num-executors est déterminé par la valeur minimale entre la contrainte de mémoire et la contrainte de processeur. 

num-executors = Min (nombre total de cœurs virtuels/nombre de cœurs par exécuteur, mémoire disponible/exécuteur-mémoire)

Définir un nombre plus élevé pour num-executors’augmente pas nécessairement les performances.  Vous devez prendre en compte le fait que l’ajout d’exécuteurs ajoute une charge pour chaque exécuteur supplémentaire, ce qui peut dégrader les performances.  Num-executors est limité par les ressources de cluster.    

## <a name="example-calculation"></a>Exemple de calcul

Supposons que vous possédez un cluster composé de 8 nœuds D4v2 exécutant 2 applications, dont celle que vous souhaitez exécuter.  

**Étape 1 : Déterminer le nombre d’applications s’exécutant sur votre cluster** : vous savez que vous avez 2 applications sur votre cluster, y compris celle que vous souhaitez exécuter.  

**Étape 2 : Configurer executor-memory** : pour cet exemple, nous déterminons que 6 Go pour executor-memory seront suffisants pour les travaux intensifs en E/S.  

executor-memory = 6 Go

**Étape 3 : Définir executor-cores** : dans la mesure où il s’agit d’un travail intensif en E/S, nous pouvons définir le nombre de cœurs pour chaque exécuteur sur 4.  Définir le nombre de cœurs par exécuteur sur une valeur supérieure à 4 peut provoquer des problèmes de garbage collection.  

executor-cores = 4

**Étape 4 : Déterminer la quantité de mémoire YARN du cluster** : nous accédons à Ambari pour déterminer que chaque D4v2 possède 25 Go de mémoire YARN.  Étant donné qu’il y a 8 nœuds, la mémoire YARN disponible est multipliée par 8.

- Mémoire YARN totale = nœuds * mémoire YARN* par nœud
- Mémoire YARN totale = 8 nœuds * 25 Go = 200 Go

**Étape 5 : Calculer num-executors** : le paramètre num-executors est déterminé par la valeur minimale entre la contrainte de mémoire et la contrainte de processeur, divisée par le nombre d’applications exécutées sur Spark.    

**Calculer la contrainte de mémoire** : la contrainte de mémoire est calculée comme la mémoire YARN totale divisée par la mémoire par exécuteur.

- Contrainte de mémoire = (total de la mémoire YARN / mémoire de l’exécuteur) / nombre d’applications
- Contrainte de mémoire = (200 Go/6 Go)/2
- Contrainte de mémoire = 16 (arrondi)

**Calculer la contrainte de processeur** : la contrainte de processeur est calculée comme le nombre total de cœurs YARN divisé par le nombre de cœurs par exécuteur.

- Cœurs YARN = nœuds dans le cluster * nombre de cœurs par nœud * 2
- Cœurs de fils = 8 nœuds * 8 cœurs par D14 * 2 = 128
- Contrainte UC = (nombre total de cœurs YARN/nombre de cœurs par exécuteur)/nombre d’applications
- Contrainte UC = (128/4) / 2
- Contrainte UC = 16

**Définir num-executors**

- num-executors = Min (contrainte de mémoire, contrainte UC)
- num-executors = Min (16, 16)
- num-executors = 16