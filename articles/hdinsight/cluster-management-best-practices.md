---
title: Bonnes pratiques pour la gestion des clusters - Azure HDInsight
description: Découvrez les bonnes pratiques à suivre pour gérer des clusters HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 12/02/2019
ms.openlocfilehash: 3a96a7116c83bf9ea2c3798c335c6cefcbdbc36d
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659486"
---
# <a name="hdinsight-cluster-management-best-practices"></a>Bonnes pratiques pour la gestion des clusters HDInsight

Découvrez les bonnes pratiques à suivre pour gérer des clusters HDInsight.

## <a name="how-do-i-create-hdinsight-clusters"></a>Comment créer des clusters HDInsight ?

| Option | Documents |
|---|---|
| Azure Data Factory | [Créer des clusters Apache Hadoop à la demande dans HDInsight avec Azure Data Factory](./hdinsight-hadoop-create-linux-clusters-adf.md) |
| Personnaliser un modèle Resource Manager | [Créer des clusters Apache Hadoop dans HDInsight avec des modèles Resource Manager](./hdinsight-hadoop-create-linux-clusters-arm-templates.md) |
| Modèles de démarrage rapide | [Modèles de démarrage rapide HDInsight](https://azure.microsoft.com/resources/templates/?term=hdinsight) |
| Exemples Azure | [Exemples Azure HDInsight](/samples/browse/?products=azure-hdinsight) |
| Portail Azure | [Créer des clusters Linux dans HDInsight à l’aide du portail Azure](./spark/apache-spark-intellij-tool-plugin.md) |
| Azure CLI | [Créer des clusters HDInsight à l’aide d’Azure CLI](./hdinsight-hadoop-create-linux-clusters-azure-cli.md) |
| Azure PowerShell | [Créer des clusters Linux dans HDInsight à l’aide d’Azure PowerShell](./hdinsight-hadoop-create-linux-clusters-azure-powershell.md) |
| cURL | [Créer des clusters Apache Hadoop à l’aide de l’API REST Azure](./hdinsight-hadoop-create-linux-clusters-curl-rest.md) |
| SDK (.NET, Python, Java) | [.NET](/dotnet/api/overview/azure/hdinsight?view=azure-dotnet&preserve-view=true), [Python](/python/api/overview/azure/hdinsight?preserve-view=true&view=azure-python), [Java](/java/api/overview/azure/hdinsight?preserve-view=true&view=azure-java-stable), [Go](./hdinsight-go-sdk-overview.md) |

> [!Note]
> Si vous créez un cluster en reprenant le nom d’un cluster existant, attendez la fin de la suppression du cluster précédent avant de créer votre cluster.

## <a name="how-do-i-customize-hdinsight-clusters"></a>Comment personnaliser des clusters HDInsight ?

| Option | Documents |
|---|---|
| Actions de script | [Personnaliser des clusters Azure HDInsight à l’aide d’actions de script](./hdinsight-hadoop-customize-cluster-linux.md) |
| Bootstrap | [Personnaliser des clusters HDInsight à l’aide de Bootstrap](./hdinsight-hadoop-customize-cluster-bootstrap.md) |
| Metastores externes | [Utiliser des magasins de métadonnées externes dans Azure HDInsight](./hdinsight-use-external-metadata-stores.md) |
| Base de données Ambari personnalisée | [Configurer des clusters HDInsight avec une base de données Ambari personnalisée](./hdinsight-custom-ambari-db.md) |

## <a name="what-are-some-errors-i-might-face-when-creating-clusters"></a>Quelles sont les erreurs susceptibles de se produire quand je crée des clusters ?

| Error | Informations complémentaires |
|---|---|
| Aucun quota | Il y a des quotas limitant le nombre de cœurs que vous pouvez créer sur votre abonnement dans chaque région. Pour plus d’informations, consultez [Planification de la capacité : quotas](./hdinsight-capacity-planning.md). |
| Pas d’autres adresses IP disponibles | Chaque réseau virtuel peut avoir un nombre limité d’adresses IP. Quand vous créez un cluster HDInsight, chaque nœud (y compris les nœuds Zookeeper et de passerelle) utilise certaines de ces adresses IP allouées. Cette erreur se produit lorsque toutes les adresses IP sont déjà utilisées.  |
| Les règles de groupe de sécurité réseau (NSG) n’autorisent pas la communication avec les fournisseurs de ressources HDInsight | Si vous utilisez des groupes de sécurité réseau ou des routes définies par l’utilisateur (UDR) pour contrôler le trafic entrant vers votre cluster HDInsight, assurez-vous que le cluster peut communiquer avec les services principaux de gestion et d’intégrité Azure. Pour plus d’informations, consultez [Étiquettes de service de groupe de sécurité réseau (NSG) pour Azure HDInsight](./hdinsight-service-tags.md) |
| Réutilisation d’un nom de cluster | Si vous réutilisez un nom de cluster existant, vous devez attendre X minutes avant de recréer le cluster. Sinon, vous verrez un message indiquant que la ressource existe déjà. |

## <a name="how-do-i-manage-running-hdinsight-clusters"></a>Comment gérer les clusters HDInsight actifs ?

| Option | Documents |
|---|---|
| Mise à l’échelle automatique | [Mettre à l’échelle automatiquement les clusters Azure HDInsight](./hdinsight-autoscale-clusters.md) |
| Mise à l’échelle manuelle | [Mettre à l’échelle des clusters Azure HDInsight](./hdinsight-scaling-best-practices.md) |
| Supervision avec Ambari| [Superviser les performances des clusters dans Azure HDInsight](./hdinsight-key-scenarios-to-monitor.md) |
| Superviser avec les journaux Azure Monitor | [Utiliser les journaux d’activité Azure Monitor pour superviser les clusters HDInsight](./hdinsight-hadoop-oms-log-analytics-tutorial.md) |
| Problèmes liés au service, maintenance planifiée et avis de sécurité et d’intégrité | [S’abonner aux alertes d’intégrité du service propres à l’abonnement](../service-health/alerts-activity-log-service-notifications-portal.md) |


## <a name="how-do-i-check-on-deleted-hdinsight-clusters"></a>Comment superviser des clusters HDInsight supprimés ?

### <a name="azure-monitor-logs"></a>Journaux d’activité Azure Monitor

Vous pouvez utiliser la requête suivante avec les journaux Azure Monitor pour superviser les clusters supprimés.

```loganalytics
AzureActivity
| where ResourceProvider == "Microsoft.HDInsight" and (OperationName == "Create or Update Cluster" or OperationName == "Delete Cluster") and ActivityStatus == "Succeeded"
```

## <a name="next-steps"></a>Étapes suivantes

* [Planification de la capacité pour les clusters HDInsight](./hdinsight-capacity-planning.md)
* [Quelles sont les configurations de nœud par défaut et recommandées pour Azure HDInsight ?](./hdinsight-supported-node-configuration.md)
