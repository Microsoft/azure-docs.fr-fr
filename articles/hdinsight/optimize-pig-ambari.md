---
title: Optimiser Apache Pig avec Apache Ambari dans Azure HDInsight
description: Utilisez l’interface utilisateur web d’Apache Ambari pour configurer et optimiser Apache Pig.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/04/2020
ms.openlocfilehash: d6f20780ccd90d83631ce07411820fb8c9280c3e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86084084"
---
# <a name="optimize-apache-pig-with-apache-ambari-in-azure-hdinsight"></a>Optimiser Apache Pig avec Apache Ambari dans Azure HDInsight

Apache Ambari est une interface web qui permet de gérer et de superviser les clusters HDInsight. Pour avoir une présentation de l’interface utilisateur web d’Ambari, consultez [Gérer des clusters HDInsight à l’aide de l’interface utilisateur web d’Apache Ambari](hdinsight-hadoop-manage-ambari.md).

Vous pouvez modifier les propriétés d’Apache Pig dans l’interface utilisateur web d’Ambari pour paramétrer les requêtes Pig. La modification directe des propriétés de Pig dans Ambari modifie les propriétés de Pig dans le fichier `/etc/pig/2.4.2.0-258.0/pig.properties`.

1. Pour modifier les propriétés de Pig, accédez à l’onglet **Configs (Configurations)** de Pig, puis développez le volet **Advanced pig-properties (Propriétés avancées de Pig)** .

1. Recherchez la propriété concernée, supprimez sa mise en commentaire et modifiez sa valeur.

1. Sélectionnez **Enregistrer** en haut à droite de la fenêtre pour enregistrer la nouvelle valeur. Certaines propriétés peuvent nécessiter un redémarrage du service.

    ![Propriétés avancées d’Apache Pig](./media/optimize-pig-ambari/advanced-pig-properties.png)

> [!NOTE]  
> Les paramètres de niveau session ont priorité sur les valeurs des propriétés dans le fichier `pig.properties`.

## <a name="tune-execution-engine"></a>Paramétrer le moteur d’exécution

Deux moteurs d’exécution sont disponibles pour exécuter des scripts Pig : MapReduce et Tez. Tez est un moteur optimisé, beaucoup plus rapide que MapReduce.

1. Pour changer de moteur d’exécution, dans le volet **Advanced pig-properties (Propriétés avancées de Pig)** , recherchez la propriété `exectype`.

1. La valeur par défaut est **MapReduce**. Remplacez-la par **Tez**.

## <a name="enable-local-mode"></a>Activer le mode local

Comme dans Hive, le mode local permet d’accélérer les tâches avec des quantités de données relativement faibles.

1. Pour activer le mode local, réglez `pig.auto.local.enabled` sur **true**. La valeur par défaut est false.

1. Les travaux avec une taille de données d’entrée inférieure à la valeur de propriété `pig.auto.local.input.maxbytes` sont considérés comme des petites tâches. La valeur par défaut est 1 Go.

## <a name="copy-user-jar-cache"></a>Copier le cache du fichier jar de l’utilisateur

Pig copie les fichiers JAR requis par les FDU dans un cache distribué pour les rendre disponibles aux nœuds de la tâche. Ces fichiers JAR ne changent pas souvent. S’il est activé, le paramètre `pig.user.cache.enabled` permet placer des fichiers JAR dans un cache afin de les réutiliser pour des travaux exécutés par le même utilisateur. Ce paramètre augmente légèrement les performances des travaux.

1. Pour l’activer, réglez `pig.user.cache.enabled` sur true. La valeur par défaut est false.

1. Pour définir le chemin de base des fichiers JAR mis en cache, définissez le chemin d’accès de base dans `pig.user.cache.location`. Par défaut, il s’agit de `/tmp`.

## <a name="optimize-performance-with-memory-settings"></a>Optimiser les performances avec les paramètres de mémoire

Les paramètres de mémoire suivants peuvent aider à optimiser les performances des scripts Pig.

* `pig.cachedbag.memusage`: Quantité maximale de mémoire affectée un conteneur. Un conteneur est une collection de tuples. Un tuple est un ensemble ordonné de champs, et un champ est une donnée. Si les données d’un conteneur dépassent la mémoire donnée, elles sont déversées sur le disque. La valeur par défaut est 0,2, soit 20 % de la mémoire disponible. Cette mémoire est partagée entre tous les conteneurs d’une application.

* `pig.spill.size.threshold`: les conteneurs supérieurs à ce seuil de dépassement de capacité (en octets) sont déversés sur le disque. La valeur par défaut est 5 Mo.

## <a name="compress-temporary-files"></a>Compresser les fichiers temporaires

Pig génère des fichiers temporaires lors de l’exécution du travail. La compression des fichiers temporaires entraîne une augmentation des performances lors de la lecture ou de l’écriture de fichiers sur le disque. Les paramètres suivants permettent de compresser les fichiers temporaires.

* `pig.tmpfilecompression`: quand ce paramètre a la valeur true, il active la compression des fichiers temporaires. La valeur par défaut est false.

* `pig.tmpfilecompression.codec`: codec à utiliser pour compresser les fichiers temporaires. Les codecs de compression recommandés sont LZO et Snappy pour leur faible utilisation du processeur.

## <a name="enable-split-combining"></a>Activer la combinaison de fractionnements

Lorsque ce paramètre est activé, les petits fichiers sont combinés pour réduire le nombre de tâches de mappage. Ce paramètre améliore l’efficacité des travaux comportant de nombreux petits fichiers. Pour l’activer, réglez `pig.noSplitCombination` sur true. La valeur par défaut est false.

## <a name="tune-mappers"></a>Paramétrer les mappeurs

Le nombre de mappeurs est contrôlé en modifiant la propriété `pig.maxCombinedSplitSize`. Cette propriété spécifie la taille des données qu'une tâche de mappage unique doit traiter. La valeur par défaut est la taille de bloc par défaut du système de fichiers. Augmenter cette valeur permet de diminuer le nombre de tâches du mappeur.

## <a name="tune-reducers"></a>Paramétrer les réducteurs

Le nombre de réducteurs est calculé en fonction du paramètre `pig.exec.reducers.bytes.per.reducer`. Le paramètre spécifie le nombre d’octets traités par réducteur (1 Go par défaut). Pour limiter le nombre maximal de réducteurs, définissez la propriété `pig.exec.reducers.max` (définie sur 999 par défaut).

## <a name="next-steps"></a>Étapes suivantes

* [Gérer des clusters HDInsight avec l’interface utilisateur web d’Ambari](hdinsight-hadoop-manage-ambari.md)
* [API REST Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md)
* [Optimiser les clusters](./hdinsight-changing-configs-via-ambari.md)
* [Optimiser Apache HBase](./optimize-hbase-ambari.md)
* [Optimiser Apache Hive](./optimize-hive-ambari.md)
