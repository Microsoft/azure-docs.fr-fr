---
title: Utiliser les journaux Azure Monitor pour surveiller les clusters Azure HDInsight
description: Découvrez comment utiliser les journaux d’activité Azure Monitor pour surveiller les travaux en cours d’exécution dans un cluster HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020, devx-track-azurecli, devx-track-azurepowershell
ms.date: 05/13/2020
ms.openlocfilehash: 775e2fad573832dd29fc45985c6d6bd0a50fdf3c
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546090"
---
# <a name="use-azure-monitor-logs-to-monitor-hdinsight-clusters"></a>Utiliser les journaux d’activité Azure Monitor pour superviser les clusters HDInsight

Découvrez comment activer les journaux Azure Monitor pour superviser les opérations de cluster Hadoop dans HDInsight et comment ajouter une solution de supervision HDInsight.

Les [journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md) sont un service Azure Monitor qui supervise vos environnements Cloud et locaux. La supervision consiste à maintenir leur disponibilité et leurs performances. Elle collecte les données générées par les ressources de votre cloud et de vos environnements locaux et d’autres outils de supervision. Les données sont utilisées pour fournir une analyse englobant plusieurs sources.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Un espace de travail Log Analytics. Considérez cet espace de travail comme un environnement des journaux d’activité Azure Monitor avec son propre référentiel de données, et ses propres sources de données et solutions. Pour obtenir des instructions, consultez la rubrique [Créer un espace de travail Log Analytics](../azure-monitor/learn/quick-collect-azurevm.md#create-a-workspace).

* Un cluster Azure HDInsight. Vous pouvez actuellement utiliser les journaux d’activité Azure Monitor avec les types de cluster HDInsight suivants :

  * Hadoop
  * hbase
  * Interactive Query
  * Kafka
  * Spark
  * Storm

  Pour savoir comment créer un cluster HDInsight, consultez [Bien démarrer avec Azure HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md).  

* Si vous utilisez PowerShell, vous aurez besoin du [module Az](/powershell/azure/). Vérifiez que vous avez la version la plus récente. Si nécessaire, exécutez `Update-Module -Name Az`.

* Si vous voulez utiliser Azure CLI et que vous ne l’avez pas encore installé, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

> [!NOTE]  
> Il est recommandé de placer le cluster HDInsight et l’espace de travail Log Analytics dans la même région pour obtenir de meilleures performances. Notez que les journaux d’activité Azure Monitor ne sont pas disponibles dans toutes les régions Azure.

## <a name="enable-azure-monitor-using-the-portal"></a>Activer Azure Monitor à l’aide du portail

Cette section vous explique comment configurer un cluster Hadoop HDInsight existant pour utiliser un espace de travail Azure Log Analytics afin de superviser les travaux, les journaux de débogage, etc.

1. Dans le [Portail Azure](https://portal.azure.com/), sélectionnez votre cluster. Le cluster est ouvert dans une nouvelle page du portail.

1. À gauche, sous **Supervision** , sélectionnez **Azure Monitor**.

1. Dans la vue principale, sous **Intégration à Azure Monitor** , sélectionnez **Activer**.

1. Dans la liste déroulante **Sélectionner un espace de travail** , sélectionnez un espace de travail Log Analytics existant.

1. Sélectionnez **Enregistrer**.  L’enregistrement du paramètre prend quelques instants.

    ![Activer la supervision pour les clusters HDInsight](./media/hdinsight-hadoop-oms-log-analytics-tutorial/azure-portal-monitoring.png "Activer la supervision pour les clusters HDInsight")

## <a name="enable-azure-monitor-using-azure-powershell"></a>Activer Azure Monitor à l’aide d’Azure PowerShell

Vous pouvez activer les journaux Azure Monitor à l’aide de l’applet de commande [Enable-AzHDInsightMonitoring](/powershell/module/az.hdinsight/enable-azhdinsightmonitoring) du module Az Azure PowerShell.

```powershell
# Enter user information
$resourceGroup = "<your-resource-group>"
$cluster = "<your-cluster>"
$LAW = "<your-Log-Analytics-workspace>"
# End of user input

# obtain workspace id for defined Log Analytics workspace
$WorkspaceId = (Get-AzOperationalInsightsWorkspace `
                    -ResourceGroupName $resourceGroup `
                    -Name $LAW).CustomerId

# obtain primary key for defined Log Analytics workspace
$PrimaryKey = (Get-AzOperationalInsightsWorkspace `
                    -ResourceGroupName $resourceGroup `
                    -Name $LAW | Get-AzOperationalInsightsWorkspaceSharedKeys).PrimarySharedKey

# Enables monitoring and relevant logs will be sent to the specified workspace.
Enable-AzHDInsightMonitoring `
    -ResourceGroupName $resourceGroup `
    -Name $cluster `
    -WorkspaceId $WorkspaceId `
    -PrimaryKey $PrimaryKey

# Gets the status of monitoring installation on the cluster.
Get-AzHDInsightMonitoring `
    -ResourceGroupName $resourceGroup `
    -Name $cluster
```

Pour désactiver, utilisez l’applet de commande [Disable-AzHDInsightMonitoring](/powershell/module/az.hdinsight/disable-azhdinsightmonitoring) :

```powershell
Disable-AzHDInsightMonitoring -Name "<your-cluster>"
```

## <a name="enable-azure-monitor-using-azure-cli"></a>Activer Azure Monitor à l’aide d’Azure CLI

Vous pouvez activer les journaux d’activité d’Azure Monitor à l’aide de la commande Azure CLI `[az hdinsight monitor enable`](/cli/azure/hdinsight/monitor#az-hdinsight-monitor-enable).

```azurecli
# set variables
export resourceGroup=RESOURCEGROUPNAME
export cluster=CLUSTERNAME
export LAW=LOGANALYTICSWORKSPACENAME

# Enable the Azure Monitor logs integration on an HDInsight cluster.
az hdinsight monitor enable --name $cluster --resource-group $resourceGroup --workspace $LAW

# Get the status of Azure Monitor logs integration on an HDInsight cluster.
az hdinsight monitor show --name $cluster --resource-group $resourceGroup
```

Pour les désactiver, utilisez la commande [`az hdinsight monitor disable`](/cli/azure/hdinsight/monitor#az-hdinsight-monitor-disable).

```azurecli
az hdinsight monitor disable --name $cluster --resource-group $resourceGroup
```

## <a name="install-hdinsight-cluster-management-solutions"></a>Installer des solutions de gestion de clusters HDInsight

HDInsight fournit des solutions de gestion de clusters que vous pouvez ajouter pour les journaux d’activité Azure Monitor. Les [solutions de gestion](../azure-monitor/insights/solutions.md) ajoutent des fonctionnalités aux journaux d’activité Azure Monitor en fournissant des données et des outils d’analyse supplémentaires. Ces solutions collectent des métriques de performance importantes de vos clusters HDInsight et fournissent les outils permettant d’explorer les métriques. Elles fournissent également des visualisations et des tableaux de bord pour la plupart des types de cluster pris en charge dans HDInsight. En vous appuyant sur les mesures collectées avec la solution, vous êtes en mesure de créer des règles et des alertes personnalisées de surveillance.

Solutions HDInsight disponibles :

* HDInsight Hadoop Monitoring
* HDInsight HBase Monitoring
* HDInsight Interactive Query Monitoring
* HDInsight Kafka Monitoring
* HDInsight Spark Monitoring
* HDInsight Storm Monitoring

Pour obtenir des instructions sur les solutions de gestion, consultez [Solutions de gestion dans Azure](../azure-monitor/insights/solutions.md#install-a-monitoring-solution). Pour faire des essais, installez une solution de supervision HDInsight Hadoop. Vous voyez alors une vignette **HDInsightHadoop** sous **Résumé**. Sélectionnez la vignette **HDInsightHadoop**. La solution HDInsightHadoop ressemble à ceci :

![Vue de la solution de supervision HDInsight](media/hdinsight-hadoop-oms-log-analytics-tutorial/hdinsight-oms-hdinsight-hadoop-monitoring-solution.png)

Étant donné que le cluster est tout nouveau, le rapport ne contient aucune activité.

## <a name="configuring-performance-counters"></a>Configuration des compteurs de performances

Azure Monitor prend en charge la collecte et l’analyse des métriques de performances pour les nœuds de votre cluster. Pour plus d’informations, consultez [Sources de données de performances Linux dans Azure Monitor](../azure-monitor/platform/data-sources-performance-counters.md#linux-performance-counters).

## <a name="cluster-auditing"></a>Audit de cluster

HDInsight prend en charge l’audit de cluster avec des journaux Azure Monitor en important les types de journaux suivants :

* `log_gateway_audit_CL` : ce tableau répertorie les journaux d’audit de nœuds de passerelle de cluster qui affichent des tentatives de connexion réussies et ayant échoué.
* `log_auth_CL` : ce tableau répertorie les journaux SSH contenant des tentatives de connexion réussies et ayant échoué.
* `log_ambari_audit_CL` : ce tableau répertorie les journaux d’audit d’Ambari.
* `log_ranger_audti_CL` : ce tableau répertorie les journaux d’audit d’Apache Ranger sur des clusters ESP.

## <a name="next-steps"></a>Étapes suivantes

* [Interroger les journaux Azure Monitor pour surveiller les clusters HDInsight](hdinsight-hadoop-oms-log-analytics-use-queries.md)
* [Superviser la disponibilité des clusters à l’aide d’Apache Ambari et des journaux Azure Monitor](./hdinsight-cluster-availability.md)