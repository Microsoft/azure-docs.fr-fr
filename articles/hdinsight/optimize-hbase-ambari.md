---
title: Optimiser Apache HBase avec Apache Ambari dans Azure HDInsight
description: Utilisez l’interface utilisateur web d’Apache Ambari pour configurer et optimiser Apache HBase.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/04/2020
ms.openlocfilehash: b262e07bd07320e4b10b12a2f2cf07b97e58c61e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91821707"
---
# <a name="optimize-apache-hbase-with-apache-ambari-in-azure-hdinsight"></a>Optimiser Apache HBase avec Apache Ambari dans Azure HDInsight

Apache Ambari est une interface web qui permet de gérer et de superviser les clusters HDInsight. Pour avoir une présentation de l’interface utilisateur web d’Ambari, consultez [Gérer des clusters HDInsight à l’aide de l’interface utilisateur web d’Apache Ambari](hdinsight-hadoop-manage-ambari.md).

La configuration d’Apache HBase est modifiable sous l’onglet **HBase Configs**. Les sections suivantes décrivent certains paramètres de configuration importants qui affectent les performances de HBase.

## <a name="set-hbase_heapsize"></a>Définir HBASE_HEAPSIZE

La taille de tas HBase spécifie la quantité maximale de tas à utiliser en mégaoctets par les serveurs de *région* et *maîtres*. La valeur par défaut est 1 000 Mo. Cette valeur doit être adaptée à la charge de travail du cluster.

1. Pour la modifier, accédez au volet **Advanced HBase-env (HBase-env avancé)** dans l’onglet **Configs (Configurations)** et recherchez le paramètre `HBASE_HEAPSIZE`.

1. Remplacez la valeur par défaut par 5 000 Mo.

    ![« Apache Ambari - Taille de segment de mémoire HBase »](./media/optimize-hbase-ambari/ambari-hbase-heapsize.png)

## <a name="optimize-read-heavy-workloads"></a>Optimiser les charges de travail nécessitant beaucoup de lectures

Les configurations suivantes sont importantes pour améliorer les performances des charges de travail nécessitant beaucoup de lectures.

### <a name="block-cache-size"></a>Taille du cache de blocs

Le cache de blocs est le cache de lecture. Sa taille est contrôlée par le paramètre `hfile.block.cache.size`. La valeur par défaut est de 0,4, soit 40 % de la mémoire totale du serveur de la région. Plus la taille du cache de blocs est importante, plus les lectures aléatoires sont rapides.

1. Pour modifier ce paramètre, accédez à l’onglet **Settings (Paramètres)** dans l’onglet **Configs (Configurations)** de HBase, puis recherchez **% of RegionServer Allocated to Read Buffers (% du serveur de région alloué aux mémoires tampons de lecture)** .

    ![Apache HBase - Taille du cache de bloc de mémoire](./media/optimize-hbase-ambari/hbase-block-cache-size.png)

1. Pour modifier la valeur, sélectionnez l’icône **Edit (Modifier)** .

### <a name="memstore-size"></a>Taille de memstore

Toutes les modifications sont stockées dans la mémoire tampon appelée *Memstore*. Cette mémoire tampon augmente la quantité totale de données qui peuvent être écrites sur le disque en une seule opération. Elle accélère également l’accès aux modifications récentes. La taille de Memstore est définie par les deux paramètres suivants :

* `hbase.regionserver.global.memstore.UpperLimit`: définit le pourcentage maximal du serveur de région que Memstore peut utiliser.

* `hbase.regionserver.global.memstore.LowerLimit`: définit le pourcentage minimal du serveur de région que Memstore peut utiliser.

Pour optimiser les lectures aléatoires, vous pouvez réduire les limites supérieure et inférieure de Memstore.

### <a name="number-of-rows-fetched-when-scanning-from-disk"></a>Nombre de lignes extraites lors de l’analyse du disque

Le paramètre `hbase.client.scanner.caching` définit le nombre de lignes lues sur le disque lorsque la méthode `next` est appelée sur un scanneur.  La valeur par défaut est 100. Plus ce nombre est élevé, moins les appels distants effectués depuis le client vers le serveur de région sont nombreux, ce qui accélère l’analyse. Toutefois, ce paramètre augmente également la sollicitation de la mémoire sur le client.

![Apache - Nombre de lignes extraites dans HBase](./media/optimize-hbase-ambari/hbase-num-rows-fetched.png)

> [!IMPORTANT]  
> Ne définissez pas une valeur qui rende le délai entre l’appel de la méthode suivante sur un scanneur supérieure au délai du scanneur. La durée d’expiration du scanneur est définie par la propriété `hbase.regionserver.lease.period`.

## <a name="optimize-write-heavy-workloads"></a>Optimiser les charges de travail nécessitant beaucoup d’écritures

Les configurations suivantes sont importantes pour améliorer les performances des charges de travail nécessitant beaucoup d’écritures.

### <a name="maximum-region-file-size"></a>Taille maximale du fichier de région

HBase stocke les données dans un format de fichier interne, appelé *HFile*. La propriété `hbase.hregion.max.filesize` définit la taille d’un fichier HFile unique pour une région.  Une région est divisée en deux si la somme de tous les fichiers HFile d’une région est supérieure à ce paramètre.

![« Apache - Taille maximale du fichier de région dans HBase »](./media/optimize-hbase-ambari/hbase-hregion-max-filesize.png)

Plus la taille du fichier de région est importante, plus le nombre de fractionnements est réduit. Vous pouvez augmenter la taille du fichier pour déterminer une valeur qui optimise les performances d’écriture.

### <a name="avoid-update-blocking"></a>Éviter le blocage des mises à jour

* La propriété `hbase.hregion.memstore.flush.size` définit la taille à laquelle Memstore est vidé sur le disque. La taille par défaut est de 128 Mo.

* Le multiplicateur de bloc de région HBase est défini par `hbase.hregion.memstore.block.multiplier`. La valeur par défaut est 4. La valeur maximale autorisée est 8 Go.

* HBase bloque les mises à jour si le paramètre Memstore est de (`hbase.hregion.memstore.flush.size` * `hbase.hregion.memstore.block.multiplier`) octets.

    Avec les valeurs par défaut de la taille de vidage et du multiplicateur de bloc, les mises à jour sont bloquées lorsque Memstore a une taille égale à 128 * 4 = 512 Mo. Pour réduire le nombre de blocages des mises à jour, augmentez la valeur de `hbase.hregion.memstore.block.multiplier`.

![Apache - Multiplicateur de bloc de région de HBase](./media/optimize-hbase-ambari/hbase-hregion-memstore-block-multiplier.png)

## <a name="define-memstore-size"></a>Définir la taille de Memstore

La taille de Memstore est définie par les paramètres `hbase.regionserver.global.memstore.upperLimit` et `hbase.regionserver.global.memstore.lowerLimit`. En définissant deux valeurs égales, vous réduisez les pauses pendant les écritures (ce qui augmente la fréquence des vidages) et améliore les performances d’écriture.

## <a name="set-memstore-local-allocation-buffer"></a>Définir la mémoire tampon d’allocation locale de Memstore

L’utilisation de la mémoire tampon d’allocation locale de Memstore est déterminée par la propriété `hbase.hregion.memstore.mslab.enabled`. Lorsqu’il est activé (true), ce paramètre empêche la fragmentation des tas pendant une opération nécessitant beaucoup d’écritures. La valeur par défaut est true.

![hbase.hregion.memstore.mslab.enabled](./media/optimize-hbase-ambari/hbase-hregion-memstore-mslab-enabled.png)

## <a name="next-steps"></a>Étapes suivantes

* [Gérer des clusters HDInsight avec l’interface utilisateur web d’Ambari](hdinsight-hadoop-manage-ambari.md)
* [API REST Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md)
* [Optimiser les clusters](./hdinsight-changing-configs-via-ambari.md)
* [Optimiser Apache Hive](./optimize-hive-ambari.md)
* [Optimiser Apache Pig](./optimize-pig-ambari.md)
