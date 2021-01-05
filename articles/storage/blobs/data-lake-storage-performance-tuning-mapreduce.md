---
title: 'Régler les performances : MapReduce, HDInsight et Azure Data Lake Storage Gen2 | Microsoft Docs'
description: Comprendre les recommandations relatives au réglage des performances des travaux Map Reduce sur Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2019
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: b95d37e1725940799750dbd3c29174d9855390d6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912924"
---
# <a name="tune-performance-mapreduce-hdinsight--azure-data-lake-storage-gen2"></a>Régler les performances : MapReduce, HDInsight et Azure Data Lake Storage Gen2

Découvrez les facteurs à prendre en compte quand vous réglez les performances des travaux MapReduce. Cet article donne plusieurs conseils de réglage des performances.

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Data Lake Storage Gen2**. Pour obtenir les instructions de création de compte, consultez [Démarrage rapide : Créer un compte de stockage Azure Data Lake Storage Gen2](../common/storage-account-create.md).
* Un **cluster Azure HDInsight** avec un accès à un compte Data Lake Storage Gen2. Consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md)
* **Utilisation de MapReduce sur HDInsight**.  Pour plus d’informations, consultez [Utilisation de MapReduce sur Hadoop sur HDInsight](../../hdinsight/hadoop/hdinsight-use-mapreduce.md)
* **Conseils de réglage des performances sur Data Lake Storage Gen2**.  Pour en savoir plus sur les concepts de performance d’ordre général, consultez [Conseils de réglage des performances de Data Lake Storage Gen2](data-lake-storage-performance-tuning-guidance.md)

## <a name="parameters"></a>Paramètres

Lors de l’exécution des travaux MapReduce, voici les paramètres que vous pouvez configurer pour améliorer les performances sur Data Lake Storage Gen2 :

* **Mapreduce.map.memory.mb** : la quantité de mémoire à allouer à chaque mappeur
* **Mapreduce.job.maps** : le nombre de tâches de mappage par travail
* **Mapreduce.reduce.memory.mb** : la quantité de mémoire à allouer à chaque réducteur
* **Mapreduce.job.reduces** : le nombre de tâches de réduction par travail

**Mapreduce.map.memory / Mapreduce.reduce.memory** : ce nombre doit être ajusté en fonction de la quantité de mémoire nécessaire pour la tâche de mappage/réduction.  Les valeurs par défaut de mapreduce.map.memory et mapreduce.reduce.memory peuvent être affichées dans Ambari via la configuration de YARN.  Dans Ambari, accédez à YARN et affichez l’onglet Configurations.  La mémoire YARN s’affiche.  

**Mapreduce.job.maps / Mapreduce.job.reduces** : détermine le nombre maximal de mappeurs ou de réducteurs à créer.  Le nombre de fractionnements détermine le nombre de mappeurs créés pour le travail MapReduce.  Par conséquent, vous pouvez obtenir moins mappeurs que demandé s’il existe moins de fractionnements que de mappeurs demandés.       

## <a name="guidance"></a>Assistance

> [!NOTE]
> Les instructions de ce document supposent que votre application est la seule application en cours d’exécution sur votre cluster.

**Étape 1 : Déterminer le nombre de travaux en cours d’exécution**

Par défaut, MapReduce utilise l’ensemble du cluster pour votre travail.  Vous pouvez utiliser une partie moindre du cluster en utilisant moins de mappeurs que de conteneurs disponibles.        

**Étape 2 : Configurer mapreduce.map.memory/mapreduce.reduce.memory**

La taille de la mémoire pour les tâches de mappage et de réduction dépend du travail en question.  Vous pouvez réduire la taille de la mémoire si vous souhaitez augmenter la simultanéité.  Le nombre de tâches exécutées simultanément varie selon le nombre de conteneurs.  En réduisant la quantité de mémoire par mappeur ou réducteur, il est possible de créer plus de conteneurs, pour permettre à plus de mappeurs ou réducteurs de s’exécuter simultanément.  Trop réduire la quantité de mémoire risque de causer des problèmes de mémoire insuffisante pour certains processus.  Si vous obtenez une erreur de segment de mémoire lors de l’exécution de votre travail, vous devez augmenter la mémoire par mappeur ou réducteur.  Vous devez prendre en compte le fait que l’ajout de conteneurs ajoute une charge pour chaque conteneur supplémentaire, ce qui peut dégrader les performances.  Une autre solution consiste à obtenir davantage de mémoire à l’aide d’un cluster qui possède une quantité supérieure de mémoire ou en augmentant le nombre de nœuds de votre cluster.  Une mémoire plus importante permettra d’utiliser plus de conteneurs, pour plus de simultanéité.  

**Étape 3 : Déterminer la mémoire YARN totale**

Pour régler mapreduce.job.maps/mapreduce.job.reduces, vous devez prendre en compte la quantité de mémoire YARN totale disponible à utiliser.  Ces informations sont disponibles dans Ambari.  Accédez à YARN et affichez l’onglet Configurations.  La mémoire YARN s’affiche dans cette fenêtre.  Vous devez multiplier la mémoire YARN par le nombre de nœuds dans votre cluster pour obtenir la mémoire YARN totale.

Mémoire YARN totale = nœuds * mémoire YARN par nœud

Si vous utilisez un cluster vide, la mémoire peut être la mémoire YARN totale pour votre cluster.  Si d’autres applications utilisent de la mémoire, vous pouvez choisir d’utiliser uniquement une partie de la mémoire de votre cluster en réduisant le nombre de mappeurs ou réducteurs au nombre de conteneurs que vous souhaitez utiliser.  

**Étape 4 : Calculer le nombre de conteneurs YARN**

Les conteneurs YARN déterminent la quantité de simultanéité disponible pour le travail.  Prenez la mémoire YARN totale et divisez-la par mapreduce.map.memory.  

\# de conteneurs YARN = total de la mémoire YARN/mapreduce.map.memory

**Étape 5 : Configurer mapreduce.job.maps/mapreduce.job.reduces**

Affectez à mapreduce.job.maps/mapreduce.job.reduces une valeur au moins égale au nombre de conteneurs disponibles.  Vous pouvez expérimenter davantage en augmentant le nombre de mappeurs et de réducteurs pour voir si vous obtenez de meilleures performances.  N’oubliez pas que les mappeurs supplémentaires ajouteront une charge, ainsi un trop grand nombre de mappeurs peut dégrader les performances.  

La planification et l’isolation de processeur sont désactivées par défaut. Par conséquent, le nombre de conteneurs YARN est donc limité par la mémoire.

## <a name="example-calculation"></a>Exemple de calcul

Supposez que vous avez un cluster composé de huit nœuds D14 et que vous souhaitez exécuter une tâche intensive en E/S.  Voici les calculs que vous devez faire :

**Étape 1 : Déterminer le nombre de travaux en cours d’exécution**

Dans cet exemple, supposez que votre travail est le seul en cours d’exécution.  

**Étape 2 : Configurer mapreduce.map.memory/mapreduce.reduce.memory**

Dans cet exemple, nous exécutons une tâche intensive en E/S et nous décidons que 3 Go de mémoire pour les tâches de mappage suffisent.

mapreduce.map.memory = 3 Go

**Étape 3 : Déterminer la mémoire YARN totale**

La mémoire totale du cluster est de 8 nœuds * 96 Go de mémoire YARN pour un D14 = 768 Go

**Étape 4 : Calculer le nombre de conteneurs YARN**

\# de conteneurs YARN = 768 Go de mémoire disponible/3 Go de mémoire = 256

**Étape 5 : Configurer mapreduce.job.maps/mapreduce.job.reduces**

mapreduce.map.jobs = 256

## <a name="examples-to-run"></a>Exemples à exécuter

Pour illustrer comment MapReduce s’exécute sur Data Lake Storage Gen2, voici un exemple de code qui a été exécuté sur un cluster avec les paramètres suivants :

* D14v2 16 nœuds
* Cluster Hadoop exécutant HDI 3.6

Pour commencer, voici quelques exemples de commandes pour exécuter MapReduce Teragen, Terasort et Teravalidate.  Vous pouvez ajuster ces commandes en fonction de vos ressources.

**Teragen**

```cmd
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 10000000000 abfs://example/data/1TB-sort-input
```

**Terasort**

```cmd
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 -Dmapreduce.job.reduces=512 -Dmapreduce.reduce.memory.mb=3072 abfs://example/data/1TB-sort-input abfs://example/data/1TB-sort-output
```

**Teravalidate**

```cmd
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapreduce.job.maps=512 -Dmapreduce.map.memory.mb=3072 abfs://example/data/1TB-sort-output abfs://example/data/1TB-sort-validate
```