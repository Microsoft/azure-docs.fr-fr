---
title: Haute disponibilité avec Apache Kafka - Azure HDInsight
description: Découvrez comment garantir une haute disponibilité avec Apache Kafka sur Azure HDInsight. Apprenez à rééquilibrer les réplicas de partition dans Kafka afin qu’ils soient répartis sur différents domaines d’erreur dans la région Azure qui contient HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 12/09/2019
ms.openlocfilehash: 4504463fa35d5fececbb2ab5895de2965ca4e647
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92534411"
---
# <a name="high-availability-of-your-data-with-apache-kafka-on-hdinsight"></a>Haute disponibilité de vos données avec Apache Kafka sur HDInsight

Découvrez comment configurer des réplicas de partition pour les rubriques Apache Kafka afin de tirer parti de la configuration du rack matériel sous-jacent. Cette configuration garantit la disponibilité des données stockées dans Kafka Apache sur HDInsight.

## <a name="fault-and-update-domains-with-apache-kafka"></a>Domaines d’erreur et de mise à jour avec Apache Kafka

Un domaine d’erreur est un regroupement logique de matériel sous-jacent dans un datacenter Azure. Chaque domaine d’erreur partage une source d’alimentation et un commutateur réseau communs. Les machines virtuelles et les disques managés mettant en œuvre les nœuds au sein d’un cluster HDInsight sont répartis dans ces domaines d’erreur. Cette architecture limite l’impact potentiel des défaillances de matériel physique.

Chaque région Azure possède un certain nombre de domaines d’erreur. Pour obtenir la liste des domaines et le nombre de domaines d’erreur qu’ils contiennent, consultez la documentation [Groupes à haute disponibilité](../../virtual-machines/availability.md#availability-sets).

> [!IMPORTANT]  
> Kafka n’est pas informé des domaines d’erreur. Lorsque vous créez une rubrique dans Kafka, ce dernier peut stocker tous les réplicas de partition dans le même domaine d’erreur. Pour résoudre ce problème, HDInsight fournit [l’outil de rééquilibrage de partitions Kafka](https://github.com/hdinsight/hdinsight-kafka-tools).

## <a name="when-to-rebalance-partition-replicas"></a>Quand rééquilibrer les réplicas de partition

Pour garantir la haute disponibilité de vos données Kafka, vous devez rééquilibrer les réplicas de partition de votre rubrique aux heures suivantes :

* Lorsqu’une rubrique ou une partition est créée

* Lorsque vous mettez à l’échelle un cluster

## <a name="replication-factor"></a>Facteur de réplication

> [!IMPORTANT]  
> Il est recommandé d’utiliser une région Azure qui contient les trois domaines d’erreur, et un facteur de réplication de 3.

Si vous devez utiliser une région qui contient uniquement deux domaines d’erreur, utilisez un facteur de réplication de 4 afin de répartir uniformément les réplicas sur les domaines d’erreur.

Pour obtenir un exemple de création de rubriques et de paramétrage du facteur de réplication, consultez le document [Démarrer avec Apache Kafka sur HDInsight](apache-kafka-get-started.md).

## <a name="how-to-rebalance-partition-replicas"></a>Comment rééquilibrer les réplicas de partition

Utilisez l’[outil de rééquilibrage de partition Apache Kafka](https://github.com/hdinsight/hdinsight-kafka-tools) pour rééquilibrer les rubriques sélectionnées. Cet outil doit être exécuté à partir d’une session SSH pour le nœud principal de votre cluster Kafka.

Pour plus d’informations sur la connexion à HDInsight avec SSH, consultez le document [Utilisation de SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="next-steps"></a>Étapes suivantes

* [Scalabilité d’Apache Kafka sur HDInsight](apache-kafka-scalability.md)
* [Mise en miroir avec Apache Kafka sur HDInsight](apache-kafka-mirroring.md)