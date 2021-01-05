---
title: Comment sélectionner la taille de machine virtuelle adaptée à votre cluster Azure HDInsight
description: Découvrez comment sélectionner la taille de machine virtuelle adaptée à votre cluster HDInsight.
keywords: tailles de machine virtuelle, tailles de cluster, configuration de cluster
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 10/09/2019
ms.openlocfilehash: d347707b0f48314dd872bc3ad34ac624817d2937
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92535312"
---
# <a name="selecting-the-right-vm-size-for-your-azure-hdinsight-cluster"></a>Sélection de la taille de machine virtuelle adaptée à votre cluster Azure HDInsight

Cet article explique comment sélectionner la taille de machine virtuelle appropriée pour les différents nœuds de votre cluster HDInsight. 

Commencez par comprendre comment les propriétés d’une machine virtuelle, telles que la puissance de traitement de l’UC, la taille de la RAM et la latence du réseau, affectent le traitement de vos charges de travail. Ensuite, réfléchissez à votre application et à la façon dont elle correspond à ce qui est optimisé pour les différentes familles de machines virtuelles. Assurez-vous que la famille de machines virtuelles que vous souhaitez utiliser est compatible avec le type de cluster que vous envisagez de déployer. Pour obtenir la liste de toutes les tailles de machines virtuelles prises en charge et recommandées pour chaque type de cluster, consultez [Configurations de nœuds prises en charge pour Azure HDInsight](hdinsight-supported-node-configuration.md). Enfin, vous pouvez utiliser un processus d’évaluation pour tester des exemples de charges de travail et déterminer la référence (SKU) la mieux adaptée à votre cas dans cette famille.

Pour plus d’informations sur la planification d’autres aspects de votre cluster, tels que la sélection d’un type de stockage ou d’une taille de cluster, consultez [Planification de la capacité pour les clusters HDInsight](hdinsight-capacity-planning.md).

## <a name="vm-properties-and-big-data-workloads"></a>Propriétés des machines virtuelles et charges de travail Big Data

La taille et le type de machine virtuelle sont déterminés par la puissance de traitement de l’UC, la taille de la RAM et la latence du réseau :

- Processeur : la taille de machine virtuelle détermine le nombre de cœurs. Plus il y a de cœurs, plus chaque nœud peut atteindre un degré élevé de calcul parallèle. En outre, certains types de machines virtuelles ont des cœurs plus rapides.

- RAM : la taille de machine virtuelle détermine également la quantité de RAM disponible sur la machine virtuelle. Pour les charges de travail qui stockent des données en mémoire pour le traitement, au lieu de lire à partir du disque, vérifiez que vos nœuds worker disposent de suffisamment de mémoire pour stocker les données.

- Réseau : pour la plupart des types de clusters, les données traitées par le cluster ne se trouvent pas sur le disque local, mais dans un service de stockage externe comme Data Lake Storage ou Stockage Azure. Prenez en compte le débit et la bande passante réseau entre la machine virtuelle du nœud et le service de stockage. La bande passante accessible à une machine virtuelle augmente généralement avec la taille. Pour plus d’informations, consultez [Tailles des machines virtuelles Linux dans Azure](../virtual-machines/sizes.md).

## <a name="understanding-vm-optimization"></a>Comprendre l’optimisation des machines virtuelles

Les familles de machines virtuelles dans Azure sont optimisées pour répondre à différents cas d’utilisation. Dans le tableau ci-dessous, vous trouverez certains des cas d’utilisation les plus populaires et les familles de machines virtuelles correspondantes.

| Type                     | Tailles           |    Description       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Entrée de gamme](../virtual-machines/sizes-general.md)          | A, Av2  | Les machines virtuelles affichent des performances d’unité centrale et des configurations de mémoire idéales pour les charges de travail de niveau d’entrée, propres au développement et au test. Elles sont économiques et offrent une solution à faible coût pour démarrer avec Azure. |
| [Usage général](../virtual-machines/sizes-general.md)          | D, DSv2, Dv2  | Ratio processeur/mémoire équilibré. Idéal pour le test et le développement, les bases de données petites à moyennes et les serveurs web au trafic faible à moyen. |
| [Optimisé pour le calcul](../virtual-machines/sizes-compute.md)        | F           | Ratio processeur/mémoire élevé. Convient pour les serveurs web au trafic moyen, les appareils réseau, les processus de traitement par lots et les serveurs d’application.        |
| [Mémoire optimisée](../virtual-machines/sizes-memory.md)         | Esv3, Ev3  | Ratio mémoire/processeur élevé. Idéal pour les serveurs de base de données relationnelle, les caches moyens à grands et l’analytique en mémoire.                 |

- Pour plus d’informations sur la tarification des instances de machine virtuelle disponibles dans les régions prises en charge par HDInsight, consultez la [Tarification de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

## <a name="cost-saving-vm-types-for-light-workloads"></a>Machines virtuelles économiques pour les charges de travail légères

Si vous avez des exigences de traitement légères, la [série F](https://azure.microsoft.com/blog/f-series-vm-size/) peut être un bon choix pour commencer à utiliser HDInsight. Affichant le coût le plus bas par heure, la série F offre le meilleur rapport prix-performances de la gamme Azure si l’on considère les unités de calcul Azure (ACU) par processeur virtuel.

Le tableau suivant décrit les types de cluster et les types de nœuds qui peuvent être créés avec les machines virtuelles de la série Fsv2.

| Type du cluster | Version | Nœud Worker | Nœud principal | Nœud Zookeeper |
|---|---|---|---|---|
| Spark | Tous | F4 et plus | non | non |
| Hadoop | Tous | F4 et plus | non | non |
| Kafka | Tous | F4 et plus | non | non |
| hbase | Tous | F4 et plus | non | non |
| LLAP | disabled | non | non | non |
| Storm | disabled | non | non | non |
| ML Service | HDI 3.6 UNIQUEMENT | F4 et plus | non | non |

Pour afficher les spécifications de chaque référence (SKU) de la série F, consultez les [Tailles de machine virtuelle de la série F](https://azure.microsoft.com/blog/f-series-vm-size/).

## <a name="benchmarking"></a>Benchmarking

Le benchmarking est le processus d’exécution de charges de travail simulées sur différentes machines virtuelles afin de mesurer la manière dont elles sont exécutées pour vos charges de travail de production. 

Pour plus d’informations sur les benchmarks pour les différentes tailles de cluster et références de machine virtuelle, consultez [Planification de la capacité de cluster dans Azure HDInsight](hdinsight-capacity-planning.md#choose-the-vm-size-and-type).

## <a name="next-steps"></a>Étapes suivantes

- [Configurations de nœuds prises en charge pour Azure HDInsight](hdinsight-supported-node-configuration.md)
- [Tailles des machines virtuelles Linux dans Azure](../virtual-machines/sizes.md)