---
title: Configurations de nœuds prises en charge pour Azure HDInsight
description: Découvrez les configurations minimales et recommandées pour les nœuds de cluster HDInsight.
keywords: tailles de machine virtuelle, tailles de cluster, configuration de cluster
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 05/14/2020
ms.openlocfilehash: 0bf5559590b66400fc4fc4dc27ea88c3522effb1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83646624"
---
# <a name="what-are-the-default-and-recommended-node-configurations-for-azure-hdinsight"></a>Quelles sont les configurations de nœud par défaut et recommandées pour Azure HDInsight ?

Cet article décrit les configurations de nœud par défaut et recommandées pour les clusters Azure HDInsight.

## <a name="default-and-minimum-recommended-node-configuration-and-virtual-machine-sizes-for-clusters"></a>Tailles de machine virtuelle et configuration de nœud minimum et par défaut et recommandées pour les clusters

Les tableaux suivants répertorient les tailles de machine virtuelle par défaut et recommandées des clusters HDInsight.  Ces informations sont nécessaires pour comprendre les tailles de machine virtuelle à utiliser lorsque vous créez des scripts PowerShell ou Azure CLI pour déployer des clusters HDInsight.

Si vous avez besoin de plus de 32 nœuds Worker dans un cluster, sélectionnez une taille de nœud principal avec au moins 8 cœurs et 14 Go de RAM.

Les seuls types de clusters dotés de disques de données sont les clusters Kafka et HBase avec la fonctionnalité Écritures accélérées activée. HDInsight prend en charge les tailles de disque P30 et S30 dans ces scénarios. Pour tous les autres types de clusters, HDInsight fournit un espace disque managé avec le cluster. À compter du 07/11/2019, la taille de disque managé de chaque nœud du cluster nouvellement créé est de 128 Go. Cela ne peut pas être modifié.

Les spécifications de tous les types de machines virtuelles recommandés minimum utilisés dans ce document sont résumées dans le tableau suivant.

| Taille              | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Débit de stockage temporaire max. : IOPS / MBps en lecture / MBps en écriture | Disques de données max. / débit : E/S par seconde | Nombre max de cartes réseau / Bande passante réseau attendue (MBps) |
|-------------------|-----------|-------------|----------------|----------------------------------------------------------|-----------------------------------|------------------------------|
| Standard_D3_v2 | 4    | 14          | 200                    | 12000 / 187 / 93                                           | 16             / 16 x 500           | 4 / 3 000                                       |
| Standard_D4_v2 | 8    | 28          | 400                    | 24000 / 375 / 187                                          | 32            / 32 x 500           | 8 / 6 000                                       |
| Standard_D5_v2 | 16   | 56          | 800                    | 48000 / 750 / 375                                          | 64             / 64 x 500           | 8 / 12000                                    |
| Standard_D12_v2   | 4         | 28          | 200            | 12000 / 187 / 93                                         | 16 / 16 x 500                         | 4 / 3 000                     |
| Standard_D13_v2   | 8         | 56          | 400            | 24000 / 375 / 187                                        | 32 / 32 x 500                       | 8 / 6 000                     |
| Standard_D14_v2   | 16        | 112         | 800            | 48000 / 750 / 375                                        | 64 / 64 x 500                       | 8 / 12000          |
| Standard_A1_v2  | 1         | 2           | 10             | 1000 / 20 / 10                                           | 2 / 2 x 500               | 2 / 250                 |
| Standard_A2_v2  | 2         | 4           | 20             | 2000 / 40 / 20                                           | 4 / 4 x 500               | 2 / 500                 |
| Standard_A4_v2  | 4         | 8           | 40             | 4000 / 80 / 40                                           | 8 / 8 x 500               | 4 / 1 000                     |

Pour plus d’informations sur les spécifications de chaque type de machine virtuelle, consultez les documents suivants :

* [Tailles de machine virtuelle à usage général : Série Dv2 1-5](../virtual-machines/dv2-dsv2-series.md)
* [Tailles de machine virtuelle à mémoire optimisée : Série Dv2 11-15](../virtual-machines/dv2-dsv2-series-memory.md)
* [Tailles de machine virtuelle à usage général : Série Av2 1-8](../virtual-machines/av2-series.md)

### <a name="all-supported-regions-except-brazil-south-and-japan-west"></a>Toutes les régions prises en charge à l’exception des régions Brésil Sud et Japon Ouest

> [!Note]
> Pour obtenir l’identificateur de référence (SKU) à utiliser dans PowerShell et d’autres scripts, ajoutez `Standard_` au début de toutes les références (SKU) de machine virtuelle dans les tableaux ci-dessous. Par exemple, `D12_v2` deviendrait `Standard_D12_v2`.

| Type de cluster | Hadoop | hbase | Interactive Query | Storm | Spark | ML Server | Kafka |
|---|---|---|---|---|---|---|---|
| Head : taille de machine virtuelle par défaut | D12_v2 | D12_v2 | D13_v2 | A4_v2 | D12_v2, <br/>D13_v2* | D12_v2 | D3_v2 |
| Head : tailles de machine virtuelle recommandées minimum | D5_v2 | D3_v2 | D13_v2 | A4_v2 | D12_v2, <br/>D13_v2* | D12_v2 | D3_v2 |
| Worker : taille de machine virtuelle par défaut | D4_v2 | D4_v2 | D14_v2 | D3_v2 | D13_v2 | D4_v2 | 4 D12_v2 avec 2 disques S30 par répartiteur |
| Worker : tailles de machine virtuelle recommandées minimum | D5_v2 | D3_v2 | D13_v2 | D3_v2 | D12_v2 | D4_v2 | D3_v2 |
| Zookeeper : taille de machine virtuelle par défaut |  | A4_v2 | A4_v2 | A4_v2 |  | A2_v2 | A4_v2 |
| ZooKeeper : tailles de machine virtuelle recommandées minimum |  | A4_v2 | A4_v2 | A2_v2 |  | A2_v2 | A4_v2 |
| ML Services : taille de machine virtuelle par défaut |  |  |  |  |  | D4_v2 |  |
| ML Services : tailles de machine virtuelle recommandées minimum |  |  |  |  |  | D4_v2 |  |

\* = Tailles de machine virtuelle pour les clusters du Pack Sécurité Entreprise (ESP) Spark

### <a name="brazil-south-and-japan-west-only"></a>Brésil Sud et Japon Ouest uniquement

| Type de cluster | Hadoop | hbase | Interactive Query | Storm | Spark | ML Services |
|---|---|---|---|---|---|---|
| Head : taille de machine virtuelle par défaut | D12 | D12 | D13 | A4_v2 | D12 | D12 |
| Head : tailles de machine virtuelle recommandées minimum | D5_v2 | D3_v2 | D13_v2 | A4_v2 | D12_v2 | D12_v2 |
| Worker : taille de machine virtuelle par défaut | D4 | D4 | D14 | D3 | D13 | D4 |
| Worker : tailles de machine virtuelle recommandées minimum | D5_v2 | D3_v2 | D13_v2 | D3_v2 | D12_v2 | D4_v2 |
| Zookeeper : taille de machine virtuelle par défaut |  | A4_v2 | A4_v2 | A4_v2 |  | A2_v2 |
| ZooKeeper : tailles de machine virtuelle recommandées minimum |  | A4_v2 | A4_v2 | A4_v2 |  | A2_v2 |
| ML Services : tailles de machine virtuelle par défaut |  |  |  |  |  | D4 |
| ML Services : tailles de machine virtuelle recommandées minimum |  |  |  |  |  | D4_v2 |

> [!NOTE]
> - Head est appelé *Nimbus* pour le type de cluster Storm.
> - L’élément Worker est appelé *Supervisor* pour le type de cluster Storm.
> - L’élément Worker est appelé *Region* pour le type de cluster HBase.

## <a name="next-steps"></a>Étapes suivantes

* [Quels sont les composants et versions Apache Hadoop disponibles avec HDInsight ?](hdinsight-component-versioning.md)
