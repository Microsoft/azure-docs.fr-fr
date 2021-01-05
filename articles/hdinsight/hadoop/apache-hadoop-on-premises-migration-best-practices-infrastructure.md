---
title: 'Infrastructure : Apache Hadoop en local vers Azure HDInsight'
description: Découvrez les bonnes pratiques sur l’infrastructure pour la migration des clusters Hadoop locaux vers Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/06/2019
ms.openlocfilehash: e412b82be911f0b4ba2e5cda51495cdcd7826917
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92542299"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---infrastructure-best-practices"></a>Migrer des clusters Apache Hadoop locaux vers Azure HDInsight - bonnes pratiques sur l’infrastructure

Cet article fournit des recommandations pour la gestion de l’infrastructure des clusters Azure HDInsight. Il fait partie d’une série qui propose des bonnes pratiques pour aider à la migration des systèmes Apache Hadoop locaux vers Azure HDInsight.

## <a name="plan-for-hdinsight-cluster-capacity"></a>Plan pour la capacité du cluster HDInsight

Les principaux choix à effectuer pour la planification de la capacité des clusters HDInsight sont les suivants :

**Région**  
La région Azure détermine où le cluster est physiquement approvisionné. Pour réduire la latence de lecture et d’écriture, le cluster doit être dans la même région que les données.

**Emplacement et taille de stockage**  
Le stockage par défaut doit être dans la même région que le cluster.  Pour un cluster à 48 nœuds, il est recommandé d’avoir 4 à 8 comptes de stockage. Bien que le stockage total actuel puisse être suffisant, chaque compte de stockage fournit une bande passante réseau supplémentaire pour les nœuds de calcul. Quand il existe plusieurs comptes de stockage, utilisez un nom aléatoire pour chaque compte de stockage, sans préfixe. L’objectif de l’attribution aléatoire des noms est de réduire le risque de goulot d’étranglement de stockage (limitation) ou les défaillances en mode commun parmi tous les comptes. Pour de meilleures performances, utilisez un seul conteneur par compte de stockage.

**Taille et type de machine virtuelle (prend désormais en charge la série G)**  
Chaque type de cluster a un ensemble de types de nœuds, et chaque type de nœud a des options spécifiques pour la taille et le type de machine virtuelle. La taille et le type de machine virtuelle sont déterminés par la puissance de traitement de l’UC, la taille de la RAM et la latence du réseau. Une charge de travail simulée peut être utilisée pour déterminer la taille de machine virtuelle optimale et le type de chaque nœud.

**Nombre de nœuds Worker**  
Le nombre initial de nœuds Worker peut être déterminé à l’aide des charges de travail simulées. Le cluster peut être mis à l’échelle par la suite en ajoutant d’autres nœuds worker pour répondre aux pics de charge. Le cluster peut ensuite revenir au niveau de départs si les nœuds Worker supplémentaires ne sont pas nécessaires.

Pour plus d’informations, consultez l’article [Planification de la capacité pour les clusters HDInsight](../hdinsight-capacity-planning.md).

## <a name="use-recommended-virtual-machine-type-for-cluster"></a>Utiliser le type de machine virtuelle recommandé pour les clusters

Consultez [Tailles des machines virtuelles et configuration des nœuds par défaut pour les clusters](../hdinsight-component-versioning.md#default-node-configuration-and-virtual-machine-sizes-for-clusters) pour connaître les types de machine virtuelle recommandés pour chaque type de cluster HDInsight.

## <a name="check-hadoop-components-availability-in-hdinsight"></a>Vérifier la disponibilité des composants Hadoop dans HDInsight

Chaque version de HDInsight est une distribution cloud d’un ensemble de composants de l’écosystème Hadoop. Consultez [Gestion des versions des composants HDInsight](../hdinsight-component-versioning.md) pour plus d’informations sur tous les composants HDInsight et leur version actuelle.

Vous pouvez également utiliser l’interface utilisateur Apache Ambari ou l’API REST Ambari pour vérifier les versions et composants Hadoop dans HDInsight.

Les applications ou composants qui étaient disponibles dans les clusters locaux mais qui ne font pas partie des clusters HDInsight peuvent être ajoutés sur un nœud de périphérie ou une machine virtuelle dans le même réseau virtuel que le cluster HDInsight. Une application Hadoop tierce qui n’est pas disponible sur Azure HDInsight peut être installée à l’aide de l’option « Applications » dans le cluster HDInsight. Des applications Hadoop personnalisées peuvent être installées sur un cluster HDInsight avec des « actions de script ». Le tableau suivant répertorie certaines des applications courantes et leurs options d’intégration HDInsight :

|**Application**|**Intégration**
|---|---|
|Ventilation|Nœud de périphérie HDInsight ou IaaS
|Alluxio|IaaS  
|Arcadia|IaaS 
|Atlas|Aucune (HDP uniquement)
|Datameer|Nœud de périphérie HDInsight
|Datastax (Cassandra)|IaaS (CosmosDB comme alternative sur Azure)
|DataTorrent|IaaS 
|Drill|IaaS 
|Ignite|IaaS
|Jethro|IaaS 
|Mapador|IaaS 
|Mongo|IaaS (CosmosDB comme alternative sur Azure)
|NiFi|IaaS 
|Presto|Nœud de périphérie HDInsight ou IaaS
|Python 2|PaaS 
|Python 3|PaaS 
|R|PaaS 
|SAS|IaaS 
|Vertica|IaaS (SQLDW comme alternative sur Azure)
|Tableau|IaaS 
|Waterline|Nœud de périphérie HDInsight
|StreamSets|Périphérie HDInsight 
|Palantir|IaaS 
|Sailpoint|Iaas 

Pour plus d’informations, consultez l’article [Composants Apache Hadoop disponibles avec différentes versions de HDInsight](../hdinsight-component-versioning.md#apache-components-available-with-different-hdinsight-versions)

## <a name="customize-hdinsight-clusters-using-script-actions"></a>Personnaliser des clusters HDInsight à l’aide d’actions de script

HDInsight fournit une méthode de configuration des clusters appelée **action de script** . Une action de script est un script bash qui s’exécute sur les nœuds dans un cluster HDInsight et peut être utilisé pour installer des composants supplémentaires et modifier des paramètres de configuration.

Les actions de script doivent être stockées sur un URI accessible à partir du cluster HDInsight. Elles peuvent être utilisées pendant ou après la création du cluster et peuvent aussi être limitées de manière à s’exécuter sur certains types de nœuds uniquement.

Le script peut être persistant ou exécuté une seule fois. Les scripts persistants servent à personnaliser les nouveaux nœuds worker ajoutés au cluster lors d’opérations de mise à l’échelle. Un script persistant peut également appliquer des modifications à un autre type de nœud, tel qu’un nœud principal, quand des opérations de mise à l’échelle ont lieu.

HDInsight propose des scripts prédéfinis pour installer les composants suivants sur des clusters HDInsight :

- Ajouter un compte de stockage Azure
- Installer Hue
- Installer Presto
- Installer Solr
- Installer Giraph
- Précharger les bibliothèques Hive
- Installer ou mettre à jour Mono

> [!Note]  
> HDInsight ne fournit pas de prise en charge directe pour les composants hadoop personnalisés ni les composants installés à l’aide d’actions de script.

Des actions de script peuvent également être publiées dans la Place de marché Azure en tant qu’application HDInsight.

Pour plus d’informations, consultez les articles suivants :

- [Installer des applications Apache Hadoop tierces sur HDInsight](../hdinsight-apps-install-applications.md)
- [Personnaliser des clusters HDInsight à l’aide d’actions de script](../hdinsight-hadoop-customize-cluster-linux.md)
- [Publier une application HDInsight sur la Place de marché Microsoft Azure](../hdinsight-apps-publish-applications.md)

## <a name="customize-hdinsight-configs-using-bootstrap"></a>Personnaliser des configurations HDInsight à l’aide de Bootstrap

Les modifications des configurations dans les fichiers config tels que `core-site.xml`, `hive-site.xml` et `oozie-env.xml` peuvent être apportées à l’aide de Bootstrap. Le script suivant est un exemple d’utilisation de l’applet de commande [New-AzHDInsightClusterConfig](/powershell/azure/new-azureps-module-az) du [module AZ](/powershell/module/az.hdinsight/new-azhdinsightcluster) PowerShell :

```powershell
# hive-site.xml configuration
$hiveConfigValues = @{"hive.metastore.client.socket.timeout"="90"}

$config = New—AzHDInsightClusterConfig '
    | Set—AzHDInsightDefaultStorage
    —StorageAccountName "$defaultStorageAccountName.blob. core.windows.net" `
    —StorageAccountKey "defaultStorageAccountKey " `
    | Add—AzHDInsightConfigValues `
        —HiveSite $hiveConfigValues

New—AzHDInsightCluster `
    —ResourceGroupName $existingResourceGroupName `
    —Cluster-Name $clusterName `
    —Location $location `
    —ClusterSizeInNodes $clusterSizeInNodes `
    —ClusterType Hadoop `
    —OSType Linux `
    —Version "3.6" `
    —HttpCredential $httpCredential `
    —Config $config
```

Pour plus d’informations, consultez l’article [Personnalisation de clusters HDInsight à l’aide de Bootstrap](../hdinsight-hadoop-customize-cluster-bootstrap.md).  Consultez également [Gérer des clusters HDInsight à l’aide de l’API REST d’Apache Ambari](../hdinsight-hadoop-manage-ambari-rest-api.md).

## <a name="access-client-tools-from-hdinsight-hadoop-cluster-edge-nodes"></a>Outils d’accès client à partir des nœuds de périphérie de clusters Hadoop HDInsight

Un nœud de périphérie vide est une machine virtuelle Linux dotée des mêmes outils clients installés et configurés que dans les nœuds principaux, mais sans services Hadoop en cours d’exécution. Le nœud de périphérie peut être utilisé aux fins suivantes :

- accès au cluster
- test des applications clientes
- hébergement des applications clientes

Les nœuds de périphérie peuvent être créés et supprimés via le portail Azure et peuvent être utilisés pendant ou après la création du cluster. Après avoir créé le nœud de périphérie, vous pouvez vous y connecter à l’aide de SSH et exécuter les outils clients pour accéder au cluster Hadoop dans HDInsight. Le point de terminaison SSH de nœud de périphérie est `<EdgeNodeName>.<ClusterName>-ssh.azurehdinsight.net:22`.


Pour plus d’informations, consultez l’article [Utiliser les nœuds de périphérie vides sur les clusters Apache Hadoop dans HDInsight](../hdinsight-apps-use-edge-node.md).


## <a name="use-scale-up-and-scale-down-feature-of-clusters"></a>Utiliser la fonctionnalité de scale-up et de scale-down des clusters

HDInsight fournit l’élasticité en vous offrant la possibilité de monter ou de descendre en puissance le nombre de nœuds de travail dans vos clusters. Cette fonctionnalité vous permet de réduire un cluster après certaines heures ou les week-ends, et de le développer pendant les pics d’activité. Pour plus d'informations, consultez les pages suivantes :

* [Mettre à l’échelle les clusters HDInsight](../hdinsight-scaling-best-practices.md).
* [Mettre à l’échelle des clusters](../hdinsight-administer-use-portal-linux.md#scale-clusters).

## <a name="use-hdinsight-with-azure-virtual-network"></a>Utiliser HDInsight avec un réseau virtuel Azure

Les réseaux virtuels Azure permettent à des ressources Azure, comme les machines virtuelles Azure, de communiquer en toute sécurité entre elles, avec Internet et avec les réseaux locaux, en filtrant et en acheminant le trafic réseau.

L’utilisation d’un réseau virtuel Azure avec HDInsight permet les scénarios suivants :

- connexion à HDInsight directement à partir d’un réseau local ;
- connexion de HDInsight à des banques de données dans un réseau virtuel Azure ;
- accès direct aux services Hadoop qui ne sont pas disponibles publiquement sur Internet, tels que les API Kafka ou l’API Java HBase.

HDInsight peut être ajouté à un réseau virtuel Azure nouveau ou existant. Si HDInsight est ajouté à un réseau virtuel existant, les groupes de sécurité réseau et itinéraires définis par l’utilisateur doivent être mis à jour pour autoriser un accès illimité à [plusieurs adresses IP](../hdinsight-management-ip-addresses.md) dans le centre de données Azure. En outre, ne bloquez pas le trafic vers les [ports](../control-network-traffic.md#required-ports) qui sont utilisés par les services HDInsight.

> [!Note]  
> HDInsight ne prend pas en charge le tunneling forcé pour l’instant. Le tunneling forcé est un paramètre de sous-réseau qui force l’acheminement du trafic Internet sortant vers un appareil à des fins d’inspection et de journalisation. Supprimez le tunneling forcé avant d’installer HDInsight dans un sous-réseau, ou créez un sous-réseau pour HDInsight. HDInsight ne prend pas non plus en charge la restriction de la connectivité réseau sortante.

Pour plus d’informations, consultez les articles suivants :

- [Vue d’ensemble des réseaux virtuels Azure](../../virtual-network/virtual-networks-overview.md)
- [Étendre HDInsight à l’aide d’un Réseau virtuel Azure](../hdinsight-plan-virtual-network-deployment.md)

## <a name="securely-connect-to-azure-services-with-azure-virtual-network-service-endpoints"></a>Se connecter en toute sécurité à des services Azure avec des points de terminaison de service de réseau virtuel Azure

HDInsight prend en charge les [points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md) qui vous permettent de vous connecter en toute sécurité aux services Stockage Blob Azure, Azure Data Lake Storage Gen2, Cosmos DB et SQL Database. En activant un point de terminaison de service pour Azure HDInsight, le trafic transite par un itinéraire sécurisé à partir du centre de données Azure. Avec ce niveau amélioré de sécurité au niveau de la couche de mise en réseau, vous pouvez verrouiller les comptes de stockage de données volumineux sur leur réseau virtuel spécifié et continuer à utiliser les clusters HDInsight sans interruption pour accéder à ces données et les traiter.

Pour plus d’informations, consultez les articles suivants :

- [Points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md)
- [Améliorer la sécurité HDInsight avec des points de terminaison de service](https://azure.microsoft.com/blog/enhance-hdinsight-security-with-service-endpoints/)

## <a name="connect-hdinsight-to-the-on-premises-network"></a>Connecter HDInsight au réseau local

HDInsight peut être connecté au réseau local à l’aide de réseaux virtuels Azure et d’une passerelle VPN. La procédure suivante peut être utilisée pour établir la connectivité :

- Utilisez HDInsight dans un réseau virtuel Azure qui dispose d’une connectivité au réseau local.
- Configurez la résolution de noms DNS entre le réseau virtuel et le réseau local.
- Configurez des groupes de sécurité réseau ou des itinéraires définis par l’utilisateur pour contrôler le trafic réseau.

Pour plus d’informations, consultez l’article [Connecter HDInsight à votre réseau local](../connect-on-premises-network.md)

## <a name="next-steps"></a>Étapes suivantes

Lisez l’article suivant de cette série : [Meilleures pratiques concernant le stockage pour une migration locale vers Azure HDInsight Hadoop](apache-hadoop-on-premises-migration-best-practices-storage.md).