---
title: Effectuer la migration d’un cluster vers une version plus récente
titleSuffix: Azure HDInsight
description: Découvrez comment effectuer la migration d’un cluster Azure HDInsight vers une version plus récente.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/31/2020
ms.openlocfilehash: f3bfd430023330d3a399a0a760fd353b6ee60941
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86085886"
---
# <a name="migrate-hdinsight-cluster-to-a-newer-version"></a>Effectuer la migration d’un cluster HDInsight vers une version plus récente

Pour tirer parti des dernières fonctionnalités proposées par HDInsight, nous vous recommandons d’effectuer régulièrement la migration des clusters HDInsight vers la version la plus récente. HDInsight ne prend pas en charge les mises à niveau sur place lorsqu’un cluster existant est mis à niveau vers une nouvelle version de composant. Vous devez créer un nouveau cluster avec la version de composant et la version de plateforme souhaitées, puis effectuer la migration de vos applications pour qu’elles utilisent le nouveau cluster. Suivez les instructions ci-dessous pour effectuer la migration de vos clusters HDInsight.

> [!NOTE]  
> Pour obtenir des informations sur les versions HDInsight prises en charge, consultez [Quels sont les différents composants Hadoop disponibles avec HDInsight ?](hdinsight-component-versioning.md#supported-hdinsight-versions).

## <a name="migration-tasks"></a>Tâches de migration

Le workflow pour mettre à niveau un cluster HDInsight est le suivant :
![Schéma du workflow de mise à niveau HDInsight](./media/hdinsight-upgrade-cluster/upgrade-workflow-diagram.png)

1. Lisez chaque section de ce document pour comprendre les modifications qui peuvent être nécessaires lors de la mise à jour de votre cluster HDInsight.
2. Créez un cluster comme environnement de test ou d’assurance qualité. Pour plus d’informations sur la création d’un cluster, consultez [Création de clusters Hadoop basés sur Linux dans HDInsight](hdinsight-hadoop-provision-linux-clusters.md).
3. Copiez les travaux, sources de données et récepteurs existants dans le nouvel environnement.
4. Effectuez des tests de validation pour vérifier que vos tâches fonctionnent comme prévu sur le nouveau cluster.

Une fois que vous avez vérifié que tout fonctionne comme prévu, planifiez un temps d’arrêt pour la migration. Pendant ce temps d’arrêt, effectuez les actions suivantes :

1. Sauvegardez toutes les données temporaires stockées localement sur les nœuds du cluster, par exemple si vous avez des données stockées directement sur un nœud principal.
1. [Supprimez le cluster existant](./hdinsight-delete-cluster.md).
1. Créez un cluster dans le même sous-réseau de réseau virtuel avec la version HDI la plus récente (ou prise en charge) à l’aide du même magasin de données par défaut que le cluster précédent a utilisé. Cela permet au nouveau cluster de continuer à travailler sur vos données de production existantes.
1. Importez toutes les données temporaires que vous avez sauvegardées.
1. Démarrez des tâches ou poursuivez le traitement avec le nouveau cluster.

## <a name="workload-specific-guidance"></a>Recommandations relatives aux charges de travail

Les documents suivants fournissent des conseils sur la migration de certaines charges de travail :

* [Migration HBase](./hbase/apache-hbase-migrate-new-version.md)
* [Migration Kafka](./kafka/migrate-versions.md)
* [Migration Hive/Interactive Query](./interactive-query/apache-hive-migrate-workloads.md)

## <a name="backup-and-restore"></a>Sauvegarde et restauration

Pour plus d’informations sur la sauvegarde et la restauration d’une base de données, consultez [Récupérer une base de données dans Azure SQL Database à l’aide des sauvegardes de base de données automatisées](../azure-sql/database/recovery-using-backups.md).

## <a name="next-steps"></a>Étapes suivantes

* [Création de clusters Hadoop basés sur Linux dans HDInsight](hdinsight-hadoop-provision-linux-clusters.md)
* [Se connecter à HDInsight à l’aide de SSH](hdinsight-hadoop-linux-use-ssh-unix.md)
* [Gérer un cluster Linux avec Apache Ambari](hdinsight-hadoop-manage-ambari.md)
* [Notes de publication HDInsight](./hdinsight-version-release.md)
