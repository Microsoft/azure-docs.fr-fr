---
title: Personnaliser les configurations de cluster Azure HDInsight à l’aide de Bootstrap
description: Apprenez à personnaliser la configuration du cluster HDInsight par programmation à l'aide de .NET, PowerShell et des modèles Resource Manager.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 04/01/2020
ms.openlocfilehash: 246c5600da3b554ba65872780f0719a58f3f4be2
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547484"
---
# <a name="customize-hdinsight-clusters-using-bootstrap"></a>Personnalisation de clusters HDInsight à l’aide de Bootstrap

Les scripts Bootstrap permettent d’installer et de configurer des composants dans Azure HDInsight par programmation.

Il existe trois approches pour définir les paramètres de fichier config à mesure que votre cluster HDInsight est créé :

* Utilisation d'Azure PowerShell
* Utilisation du Kit de développement logiciel (SDK) .NET
* Utilisation d’un modèle Azure Resource Manager

Par exemple, à l’aide de ces méthodes de programmation, vous pouvez configurer des options dans ces fichiers :

* clusterIdentity.xml
* core-site.xml
* gateway.xml
* hbase-env.xml
* hbase-site.xml
* hdfs-site.xml
* hive-env.xml
* hive-site.xml
* mapred-site
* oozie-site.xml
* oozie-env.xml
* storm-site.xml
* tez-site.xml
* webhcat-site.xml
* yarn-site.xml
* Server.Properties (configuration de répartiteurs kafka)

Pour plus d’informations sur l’installation de composants supplémentaires sur un cluster HDInsight pendant le processus de création, consultez [Personnaliser des clusters Azure HDInsight à l’aide des actions de script](hdinsight-hadoop-customize-cluster-linux.md).

## <a name="prerequisites"></a>Prérequis

* Si vous utilisez PowerShell, vous aurez besoin du [module Az](/powershell/azure/).

## <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell

Le code PowerShell suivant permet de personnaliser une configuration [Apache Hive](https://hive.apache.org/) :

> [!IMPORTANT]  
> Le paramètre `Spark2Defaults` peut être utilisé avec [Add-AzHDInsightConfigValue](/powershell/module/az.hdinsight/add-azhdinsightconfigvalue). Vous pouvez transmettre les valeurs vides pour le paramètre, comme indiqué dans l’exemple de code ci-dessous.

```powershell
# hive-site.xml configuration
$hiveConfigValues = @{ "hive.metastore.client.socket.timeout"="90s" }

$config = New-AzHDInsightClusterConfig `
    | Set-AzHDInsightDefaultStorage `
        -StorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
        -StorageAccountKey $defaultStorageAccountKey `
    | Add-AzHDInsightConfigValue `
        -HiveSite $hiveConfigValues `
        -Spark2Defaults @{}

New-AzHDInsightCluster `
    -ResourceGroupName $existingResourceGroupName `
    -ClusterName $clusterName `
    -Location $location `
    -ClusterSizeInNodes $clusterSizeInNodes `
    -ClusterType Hadoop `
    -OSType Linux `
    -Version "3.6" `
    -HttpCredential $httpCredential `
    -Config $config
```

[L’annexe](#appendix-powershell-sample) fournit un script PowerShell opérationnel complet.

**Pour vérifier la modification :**

1. Accédez à `https://CLUSTERNAME.azurehdinsight.net/`, où `CLUSTERNAME` est le nom de votre cluster.
1. Dans le menu de gauche, accédez à **Hive** > **Configuration** >  **Avancé** .
1. Développez **le site hive avancé** .
1. Recherchez **hive.metastore.client.socket.timeout** et confirmez que la valeur est **Années 90** .

Et d’autres exemples sur la personnalisation d’autres fichiers de configuration :

```xml
# hdfs-site.xml configuration
$HdfsConfigValues = @{ "dfs.blocksize"="64m" } #default is 128MB in HDI 3.0 and 256MB in HDI 2.1

# core-site.xml configuration
$CoreConfigValues = @{ "ipc.client.connect.max.retries"="60" } #default 50

# mapred-site.xml configuration
$MapRedConfigValues = @{ "mapreduce.task.timeout"="1200000" } #default 600000

# oozie-site.xml configuration
$OozieConfigValues = @{ "oozie.service.coord.normal.default.timeout"="150" }  # default 120
```

## <a name="use-net-sdk"></a>Utilisation du Kit de développement logiciel (SDK) .NET

Consultez [SDK Azure HDInsight pour .NET](/dotnet/api/overview/azure/hdinsight?view=azure-dotnet&preserve-view=true).

## <a name="use-resource-manager-template"></a>Utilisation d’un modèle Resource Manager

Vous pouvez utiliser Bootstrap dans un modèle Resource Manager :

```json
"configurations": {
    "hive-site": {
        "hive.metastore.client.connect.retry.delay": "5",
        "hive.execution.engine": "mr",
        "hive.security.authorization.manager": "org.apache.hadoop.hive.ql.security.authorization.DefaultHiveAuthorizationProvider"
    }
}
```

![Hadoop personnalise le modèle Azure Resource Manager de démarrage de cluster](./media/hdinsight-hadoop-customize-cluster-bootstrap/hdinsight-customize-cluster-bootstrap-arm.png)

Exemple d'extrait de modèle Resource Manager permettant de modifier la configuration de spark2-defaults afin de nettoyer régulièrement les journaux des événements de l'espace de stockage.  

```json
"configurations": {
    "spark2-defaults": {
        "spark.history.fs.cleaner.enabled": "true",
        "spark.history.fs.cleaner.interval": "7d",
        "spark.history.fs.cleaner.maxAge": "90d"
    }
}
```

## <a name="see-also"></a>Voir aussi

* [Créer des clusters Apache Hadoop dans HDInsight](hdinsight-hadoop-provision-linux-clusters.md) pour obtenir des instructions sur la création d’un cluster HDInsight à l’aide d’autres options personnalisées.
* [Développer des scripts d’action de script pour HDInsight](hdinsight-hadoop-script-actions-linux.md)
* [Installer et utiliser Apache Spark sur les clusters HDInsight](spark/apache-spark-jupyter-spark-sql-use-portal.md)
* [Installer et utiliser Apache Giraph sur les clusters HDInsight](./hdinsight-hadoop-hue-linux.md).

## <a name="appendix-powershell-sample"></a>Annexe : Exemple de code PowerShell

Ce script PowerShell permet de créer un cluster HDInsight et de personnaliser un paramètre Hive. Veillez à entrer des valeurs pour `$nameToken`, `$httpPassword` et `$sshPassword`.

```powershell
####################################
# Set these variables
####################################
#region - used for creating Azure service names
$nameToken = "<ENTER AN ALIAS>"
#endregion

#region - cluster user accounts
$httpUserName = "admin"  #HDInsight cluster username
$httpPassword = '<ENTER A PASSWORD>'

$sshUserName = "sshuser" #HDInsight ssh user name
$sshPassword = '<ENTER A PASSWORD>'
#endregion

####################################
# Service names and varialbes
####################################
#region - service names
$namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

$resourceGroupName = $namePrefix + "rg"
$hdinsightClusterName = $namePrefix + "hdi"
$defaultStorageAccountName = $namePrefix + "store"
$defaultBlobContainerName = $hdinsightClusterName

$location = "East US"
#endregion


####################################
# Connect to Azure
####################################
#region - Connect to Azure subscription
Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

#endregion

#region - Create an HDInsight cluster
####################################
# Create dependent components
####################################
Write-Host "Creating a resource group ..." -ForegroundColor Green
New-AzResourceGroup `
    -Name  $resourceGroupName `
    -Location $location

Write-Host "Creating the default storage account and default blob container ..."  -ForegroundColor Green
New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableHttpsTrafficOnly 1

# Note: Storage account kind BlobStorage cannot be used as primary storage.

$defaultStorageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $defaultStorageAccountName)[0].Value

$defaultStorageContext = New-AzStorageContext `
                                -StorageAccountName $defaultStorageAccountName `
                                -StorageAccountKey $defaultStorageAccountKey

New-AzStorageContainer `
    -Name $defaultBlobContainerName `
    -Context $defaultStorageContext #use the cluster name as the container name

####################################
# Create a configuration object
####################################
$hiveConfigValues = @{"hive.metastore.client.socket.timeout"="90s"}

$config = New-AzHDInsightClusterConfig `
    | Set-AzHDInsightDefaultStorage `
        -StorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
        -StorageAccountKey $defaultStorageAccountKey `
    | Add-AzHDInsightConfigValue `
        -HiveSite $hiveConfigValues `
        -Spark2Defaults @{}

####################################
# Create an HDInsight cluster
####################################
$httpPW = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
$httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$httpPW)

$sshPW = ConvertTo-SecureString -String $sshPassword -AsPlainText -Force
$sshCredential = New-Object System.Management.Automation.PSCredential($sshUserName,$sshPW)

New-AzHDInsightCluster `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $hdinsightClusterName `
    -Location $location `
    -ClusterSizeInNodes 1 `
    -ClusterType Hadoop `
    -OSType Linux `
    -Version "3.6" `
    -HttpCredential $httpCredential `
    -SshCredential $sshCredential `
    -Config $config

####################################
# Verify the cluster
####################################
Get-AzHDInsightCluster `
    -ClusterName $hdinsightClusterName

#endregion
```