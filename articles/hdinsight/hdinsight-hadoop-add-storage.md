---
title: Ajouter des comptes Stockage Azure supplémentaires à HDInsight
description: Découvrez comment ajouter des comptes Stockage Azure supplémentaires à un cluster HDInsight existant.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 04/27/2020
ms.openlocfilehash: 51977c00dc8c9932def89d54ec1b6ec34afad652
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92541993"
---
# <a name="add-additional-storage-accounts-to-hdinsight"></a>Ajouter des comptes de stockage supplémentaires à HDInsight

Découvrez comment utiliser des actions de script pour ajouter des *comptes* Stockage Azure supplémentaires à HDInsight. Les étapes décrites dans ce document permettent d’ajouter un *compte* de stockage à un cluster HDInsight existant. Cet article s’applique aux *comptes* de stockage (différents du compte de stockage en cluster par défaut) et pas à un stockage supplémentaire comme [`Azure Data Lake Storage Gen1`](hdinsight-hadoop-use-data-lake-storage-gen1.md) et [`Azure Data Lake Storage Gen2`](hdinsight-hadoop-use-data-lake-storage-gen2.md).

> [!IMPORTANT]  
> Les informations contenues dans ce document portent sur l’ajout d’un ou de plusieurs comptes de stockage supplémentaires à un cluster après sa création. Pour plus d’informations sur l’ajout de comptes de stockage lors de la création du cluster, consultez [Configurer des clusters dans HDInsight avec Apache Hadoop, Apache Spark, Apache Kafka, etc](hdinsight-hadoop-provision-linux-clusters.md).

## <a name="prerequisites"></a>Prérequis

* Un cluster Hadoop sur HDInsight. Consultez [Bien démarrer avec HDInsight sur Linux](./hadoop/apache-hadoop-linux-tutorial-get-started.md).
* Nom et clé du compte de stockage. Consultez [Gérer les clés d’accès au compte de stockage](../storage/common/storage-account-keys-manage.md).
* Si vous utilisez PowerShell, vous avez besoin du module AZ.  Consultez [Vue d’ensemble d’Azure PowerShell](/powershell/azure/).

## <a name="how-it-works"></a>Fonctionnement

Pendant le traitement, le script effectue les opérations suivantes :

* Si le compte de stockage existe déjà dans la configuration core-site.xml du cluster, le script s’arrête et aucune action supplémentaire n’est effectuée.

* Il vérifie que le compte de stockage existe et est accessible à l’aide de la clé.

* Il chiffre la clé à l’aide des informations d’identification du cluster.

* Il ajoute le compte de stockage dans le fichier core-site.xml.

* Arrête et redémarre les services Apache Oozie, Apache Hadoop YARN, Apache Hadoop MapReduce2 et Apache Hadoop HDFS. L’arrêt et le redémarrage de ces services leur permettent d’utiliser le nouveau compte de stockage.

> [!WARNING]  
> L’utilisation d’un compte de stockage dans un autre emplacement que le cluster HDInsight n’est pas prise en charge.

## <a name="add-storage-account"></a>Ajout d’un compte de stockage

Utilisez une [action de script](hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster) pour appliquer les modifications avec les considérations suivantes :

|Propriété | Valeur |
|---|---|
|URI de script bash|`https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh`|
|Type(s) de nœud|Head|
|Paramètres|`ACCOUNTNAME` `ACCOUNTKEY` `-p` (facultatif)|

* `ACCOUNTNAME` est le nom du compte de stockage à ajouter au cluster HDInsight.
* `ACCOUNTKEY` est la clé d’accès pour `ACCOUNTNAME`.
* `-p` est facultatif. si ce paramètre est spécifié, la clé n’est pas chiffrée et est stockée en texte brut dans le fichier core-site.xml.

## <a name="verification"></a>Vérification

Lorsque vous affichez le cluster HDInsight sur le Portail Azure, les comptes de stockage ajoutés par cette action de script ne s’affichent pas quand vous sélectionnez l’entrée __Comptes de stockage__ sous __Propriétés__ . Azure PowerShell et Azure CLI n’affichent pas non plus les comptes de stockage supplémentaires. Les informations de stockage n’apparaissent pas, car le script modifie uniquement la configuration `core-site.xml` du cluster. Ces informations ne sont pas utilisées pour récupérer les informations du cluster avec les API de gestion Azure.

Pour vérifier le stockage supplémentaire, appliquez l’une des méthodes indiquées ci-dessous :

### <a name="powershell"></a>PowerShell

Le script retournera les noms des comptes de stockage associés au cluster donné. Remplacez `CLUSTERNAME` par le nom de cluster réel, puis exécutez le script.

```powershell
# Update values
$clusterName = "CLUSTERNAME"

$creds = Get-Credential -UserName "admin" -Message "Enter the cluster login credentials"

$clusterName = $clusterName.ToLower();

# getting service_config_version
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName`?fields=Clusters/desired_service_config_versions/HDFS" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content

$configVersion=$respObj.Clusters.desired_service_config_versions.HDFS.service_config_version

$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=$configVersion" `
    -Credential $creds
$respObj = ConvertFrom-Json $resp.Content

# extract account names
$value = ($respObj.items.configurations | Where type -EQ "core-site").properties | Get-Member -membertype properties | Where Name -Like "fs.azure.account.key.*"
foreach ($name in $value ) { $name.Name.Split(".")[4]}
```

### <a name="apache-ambari"></a>Apache Ambari

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`, où `CLUSTERNAME` est le nom de votre cluster.

1. Accédez à **HDFS** > **Configurations** > **Avancé** > **Configuration core-site personnalisée** .

1. Observez les clés qui commencent par `fs.azure.account.key`. Le nom du compte fera partie de la clé, comme illustré dans cet exemple :

   ![vérification via Apache Ambari](./media/hdinsight-hadoop-add-storage/apache-ambari-verification.png)

## <a name="remove-storage-account"></a>Supprimer le compte de stockage

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`, où `CLUSTERNAME` est le nom de votre cluster.

1. Accédez à **HDFS** > **Configurations** > **Avancé** > **Configuration core-site personnalisée** .

1. Supprimez les clés suivantes :
    * `fs.azure.account.key.<STORAGE_ACCOUNT_NAME>.blob.core.windows.net`
    * `fs.azure.account.keyprovider.<STORAGE_ACCOUNT_NAME>.blob.core.windows.net`

Après avoir supprimé ces clés et enregistré la configuration, vous devez redémarrer Oozie, Yarn, MapReduce2, HDFS et Hive l’un après l’autre.

## <a name="known-issues"></a>Problèmes connus

### <a name="storage-firewall"></a>Pare-feu de stockage

Si vous choisissez de sécuriser votre compte de stockage à l’aide des restrictions de **pare-feu et réseaux virtuels** sur des **réseaux sélectionnés** , veillez à activer l’exception **Autoriser les services approuvés de Microsoft...** afin que HDInsight puisse accéder à votre compte de stockage`.`

### <a name="unable-to-access-storage-after-changing-key"></a>Impossible d’accéder au stockage après avoir modifié la clé

Si vous modifiez la clé d’un compte de stockage, HDInsight ne peut plus accéder au compte de stockage. HDInsight utilise une copie de la clé mise en cache dans le fichier core-site.xml pour le cluster. Cette copie mise en cache doit être mise à jour pour correspondre à la nouvelle clé.

Si vous exécutez de nouveau l’action de script, la clé n’est **pas** mise à jour, car le script vérifie s’il existe déjà une entrée pour le compte de stockage. S’il existe déjà une entrée, aucune modification n’est apportée.

Pour contourner ce problème :  
1. Supprimez le compte de stockage.
1. Ajoutez le compte de stockage.

> [!IMPORTANT]  
> La rotation de la clé de stockage pour le compte de stockage principal attaché à un cluster n’est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à ajouter des comptes de stockage supplémentaires à un cluster HDInsight existant. Pour plus d’informations sur les actions de script, consultez [Personnalisation de clusters HDInsight basés sur Linux à l’aide d’une d’action de script](hdinsight-hadoop-customize-cluster-linux.md).