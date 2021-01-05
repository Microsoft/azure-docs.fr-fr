---
title: Configurer des clusters dans HDInsight avec Apache Hadoop, Apache Spark, Apache Kafka, etc.
description: Configurez Hadoop, Kafka, Spark, HBase, R Server ou les clusters Storm pour HDInsight à partir d’un navigateur, d’Azure Classic CLI, d’Azure PowerShell, de REST ou du SDK.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017,seodec18
ms.date: 08/06/2020
ms.openlocfilehash: 6ce5de354583da04905f9f889cfabe36e6da6667
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546124"
---
# <a name="set-up-clusters-in-hdinsight-with-apache-hadoop-apache-spark-apache-kafka-and-more"></a>Configurer des clusters dans HDInsight avec Apache Hadoop, Apache Spark, Apache Kafka, etc.

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Découvrez comment installer et configurer Apache Hadoop, Apache Spark, Apache Kafka, Interactive Query, Apache HBase, ML Services ou Apache Storm dans HDInsight. En outre, découvrez comment personnaliser des clusters et ajouter une sécurité en les joignant à un domaine.

Un cluster Hadoop se compose de plusieurs machines virtuelles (nœuds) utilisées pour le traitement distribué de tâches. Azure HDInsight prend en charge les détails de mise en œuvre de l’installation et de la configuration des nœuds individuels, de sorte que vous n’avez plus qu’à fournir les informations de configuration générales.

> [!IMPORTANT]  
> La facturation du cluster HDInsight démarre à la création du cluster et s’arrête à sa suppression. La facturation est effectuée au prorata des minutes écoulées. Par conséquent, vous devez toujours supprimer votre cluster lorsqu’il n’est plus utilisé. Apprenez comment [supprimer un cluster.](hdinsight-delete-cluster.md)

Si vous utilisez plusieurs clusters ensemble, vous devrez créer un réseau virtuel et, si vous utilisez un cluster Spark, vous devrez également utiliser Hive Warehouse Connector. Pour plus d’informations, consultez [Planifier un réseau virtuel pour Azure HDInsight](./hdinsight-plan-virtual-network-deployment.md) et [Intégrer Apache Spark et Apache Hive à Hive Warehouse Connector](interactive-query/apache-hive-warehouse-connector.md).

## <a name="cluster-setup-methods"></a>Méthodes de configuration du cluster

La table suivante présente les différentes méthodes que vous pouvez utiliser pour configurer un cluster HDInsight.

| Clusters créés avec | un navigateur Web | Ligne de commande | API REST | Kit SDK |
| --- |:---:|:---:|:---:|:---:|
| [Azure portal](hdinsight-hadoop-create-linux-clusters-portal.md) |✔ |&nbsp; |&nbsp; |&nbsp; |
| [Azure Data Factory](hdinsight-hadoop-create-linux-clusters-adf.md). |✔ |✔ |✔ |✔ |
| [Azure CLI](hdinsight-hadoop-create-linux-clusters-azure-cli.md) |&nbsp; |✔ |&nbsp; |&nbsp; |
| [Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md) |&nbsp; |✔ |&nbsp; |&nbsp; |
| [cURL](hdinsight-hadoop-create-linux-clusters-curl-rest.md) |&nbsp; |✔ |✔ |&nbsp; |
| [Modèles Microsoft Azure Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md) |&nbsp; |✔ |&nbsp; |&nbsp; |

Cet article vous guide à travers la configuration du [portail Azure](https://portal.azure.com), où vous pouvez créer un cluster HDInsight.

## <a name="basics"></a>Concepts de base

![hdinsight - options de création personnalisée - création rapide](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-basics-blank-fs.png)

### <a name="project-details"></a>Détails du projet

[Azure Resource Manager](../azure-resource-manager/management/overview.md) vous permet de manipuler les ressources de votre application sous la forme d’un groupe, nommé [groupe de ressources](../azure-resource-manager/management/overview.md#resource-groups) Azure. Vous pouvez déployer, mettre à jour, surveiller ou supprimer toutes les ressources de votre application dans le cadre d’une opération unique et coordonnée.

### <a name="cluster-details"></a>Détails du cluster

#### <a name="cluster-name"></a>Nom du cluster

Les noms des clusters HDInsight présentent les restrictions suivantes :

* Caractères autorisés : a-z, 0-9, A-Z
* Longueur maximale : 59
* Noms réservés : apps
* L’étendue de l’affectation de noms de cluster porte sur l’ensemble d’Azure et sur l’ensemble des abonnements. Le nom du cluster doit donc être unique dans le monde.
* Les six premiers caractères doivent être uniques au sein d’un réseau virtuel

#### <a name="region"></a>Région

Il n’est pas nécessaire de spécifier explicitement l’emplacement du cluster : Le cluster se trouve au même emplacement que le stockage par défaut. Pour obtenir la liste des régions prises en charge, sélectionnez la liste déroulante **Région** de la rubrique [Tarification HDInsight](https://go.microsoft.com/fwLink/?LinkID=282635&clcid=0x409).

#### <a name="cluster-type"></a>Type de cluster

Actuellement, Azure HDInsight propose les types de clusters suivants, chacun avec un ensemble de composants fournissant certaines fonctionnalités.

> [!IMPORTANT]  
> Les clusters HDInsight sont disponibles dans différents types, chacun d’eux pour une charge de travail ou une technologie unique. Il n’existe aucune méthode prise en charge pour créer un cluster combinant plusieurs types, tels que Storm et HBase sur un seul cluster. Si votre solution nécessite des technologies qui sont réparties sur plusieurs types de clusters HDInsight, un [réseau virtuel Azure](../virtual-network/index.yml) peut connecter les types de cluster requis.

| Type de cluster | Fonctionnalités |
| --- | --- |
| [Hadoop](hadoop/apache-hadoop-introduction.md) |Requête par lot et analyse des données stockées |
| [HBase](hbase/apache-hbase-overview.md) |Traitement de grandes quantités de données de NoSQL sans schéma |
| [Interactive Query](./interactive-query/apache-interactive-query-get-started.md) |Mise en cache pour les requêtes Hive interactives et plus rapides |
| [Kafka](kafka/apache-kafka-introduction.md) | Plateforme de diffusion en continu distribuée qui permet de générer des pipelines de données et des applications de diffusion en continu en temps réel |
| [ML Services](r-server/r-server-overview.md) |Différentes statistiques Big Data, modélisation prédictive et fonctionnalités Machine Learning |
| [Spark](spark/apache-spark-overview.md) |Traitement en mémoire, requêtes interactives, traitement du flux de traitement micro-batch |
| [Storm](storm/apache-storm-overview.md) |Traitement d’événements en temps réel |

#### <a name="version"></a>Version

Choisissez la version de HDInsight pour ce cluster. Pour plus d’informations, voir [Versions de HDInsight prises en charge](hdinsight-component-versioning.md#supported-hdinsight-versions).

### <a name="cluster-credentials"></a>Informations d’identification du cluster

Les clusters HDInsight vous permettent de configurer deux comptes d’utilisateur lors de la création :

* Nom d’utilisateur de connexion au cluster : Le nom d’utilisateur par défaut est *admin* . Il utilise la configuration de base sur le portail Azure. Parfois, le nom par défaut est « Utilisateur du cluster » ou « Utilisateur HTTP ».
* Nom d’utilisateur SSH (Secure Shell) : sert à se connecter au cluster à l’aide de SSH. Pour en savoir plus, voir [Utilisation de SSH avec Hadoop Linux sur HDInsight depuis Linux, Unix ou OS X](hdinsight-hadoop-linux-use-ssh-unix.md).

Le nom d’utilisateur HTTP présente les restrictions suivantes :

* Caractères spéciaux autorisés : `_` et `@`
* Caractères non autorisés :  #;."',\/:`!*?$(){}[]<>|&--=+%~^espace
* Longueur maximale : 20

Le nom d’utilisateur SSH présente les restrictions suivantes :

* Caractères spéciaux autorisés : `_` et `@`
* Caractères non autorisés :  #;."',\/:`!*?$(){}[]<>|&--=+%~^espace
* Longueur maximale : 64
* Noms réservés : hadoop, users, oozie, hive, mapred, ambari-qa, zookeeper, tez, hdfs, sqoop, yarn, hcat, ams, hbase, storm, administrator, admin, user, user1, test, user2, test1, user3, admin1, 1, 123, a, actuser, adm, admin2, aspnet, backup, console, david, guest, john, owner, root, server, sql, support, support_388945a0, sys, test2, test3, user4, user5, spark

## <a name="storage"></a>Stockage

![Paramètres de stockage du cluster : Points de terminaison compatibles HDFS](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-storage.png)

Bien qu’une installation locale de Hadoop utilise le système de fichiers distribués Hadoop (HDFS) pour le stockage sur le cluster, dans le cloud, vous utilisez des points de terminaison de stockage connectés au cluster. L’utilisation de stockage cloud signifie que vous pouvez supprimer sans risque les clusters HDInsight utilisés pour le calcul tout en conservant vos données.

Les clusters HDInsight peuvent utiliser les options de stockage suivantes :

* Azure Data Lake Storage Gen2
* Azure Data Lake Storage Gen1
* Stockage Azure Usage général v2
* Stockage Azure Usage général v1
* Objet blob de blocs Stockage Azure ( **pris en charge uniquement comme stockage secondaire** )

Pour plus d’informations sur les options de stockage avec HDInsight, voir [Comparer les options de stockage à utiliser avec les clusters Azure HDInsight](hdinsight-hadoop-compare-storage-options.md).

> [!WARNING]  
> L’utilisation d’un compte de stockage supplémentaire dans un autre emplacement que le cluster HDInsight n’est pas prise en charge.

Lors de la configuration, pour le point de terminaison de stockage par défaut, vous spécifiez un conteneur blob d’un compte Stockage Azure ou Data Lake Storage. Le stockage par défaut contient les journaux des applications et du système. Vous pouvez éventuellement spécifier des comptes Stockage Azure supplémentaires et des comptes Data Lake Storage auxquels le cluster peut accéder. Le cluster HDInsight et les comptes de stockage dépendants doivent être situés au même emplacement Azure.

[!INCLUDE [secure-transfer-enabled-storage-account](../../includes/hdinsight-secure-transfer.md)]

> [!IMPORTANT]
> L’activation du transfert de stockage sécurisé après la création d’un cluster peut entraîner des erreurs lors de l’utilisation de votre compte de stockage, et n’est donc pas recommandée. Il est préférable de créer un cluster à l’aide d’un compte de stockage sur lequel le transfert sécurisé est déjà activé.

> [!Note]  
> Azure HDInsight ne transfère pas, ne déplace pas ou ne copie pas automatiquement vos données stockées dans Stockage Azure d’une région à l’autre.

### <a name="metastore-settings"></a>Paramètres du metastore

Vous pouvez créer des metastores Hive ou Apache Oozie facultatifs. Toutefois, tous les types de clusters ne prennent pas en charge les metastores et Azure Synapse Analytics n’est pas compatible avec les metastores.

Pour plus d’informations, consultez [Utiliser des magasins de métadonnées externes dans Azure HDInsight](./hdinsight-use-external-metadata-stores.md).

> [!IMPORTANT]  
> Lorsque vous créez un metastore personnalisé, n’utilisez pas un nom de base de données contenant des traits d’union, des tirets ou des espaces. Cela risque d’entraîner l’échec du processus de création du cluster.

#### <a name="sql-database-for-hive"></a>Base de données SQL pour Hive

Nous vous recommandons d’utiliser un metastore personnalisé si vous souhaitez conserver vos tables Hive après la suppression de votre cluster HDInsight. Vous pourrez joindre ce metastore à un autre cluster HDInsight.

Un metastore HDInsight créé pour une version de cluster HDInsight ne peut pas être partagé entre différentes versions de cluster HDInsight. Pour obtenir la liste des versions de HDInsight, consultez la section [Versions de HDInsight prises en charge](hdinsight-component-versioning.md#supported-hdinsight-versions).

> [!IMPORTANT]
> Le metastore par défaut fournit une Azure SQL Database avec une **limite de 5 DTU de niveau De base (sans mise à niveau possible)**  ! Adapté aux tests de base. Pour les charges de travail de production ou volumineuses, nous vous recommandons de migrer vers un metastore externe.

#### <a name="sql-database-for-oozie"></a>Base de données SQL pour Oozie

Pour améliorer les performances avec Oozie, utilisez un metastore personnalisé. Un metastore permet également d’accéder aux données de travail Oozie après la suppression de votre cluster.

#### <a name="sql-database-for-ambari"></a>Base de données SQL pour Ambari

Ambari permet de superviser les clusters HDInsight, d’apporter des modifications à la configuration et de stocker des informations de gestion de cluster ainsi que l’historique des travaux. La fonctionnalité Ambari DB personnalisée vous permet de déployer un nouveau cluster et de configurer Ambari dans une base de données externe que vous gérez. Pour plus d’informations, consultez [Base de données Ambari personnalisée](./hdinsight-custom-ambari-db.md).

> [!IMPORTANT]  
> Vous ne pouvez pas réutiliser un metastore Oozie personnalisé. Pour utiliser un metastore Oozie personnalisé, vous devez fournir une base de données Azure SQL vide lors de la création du cluster HDInsight.

## <a name="security--networking"></a>Sécurité + mise en réseau

![hdinsight options de création choisir le package de sécurité d’entreprise](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-security-networking.png)

### <a name="enterprise-security-package"></a>Package de sécurité d’entreprise

Pour les types de cluster Hadoop, Spark, HBase, Kafka et Interactive Query, vous pouvez choisir d’activer le **Pack Sécurité Entreprise** . Vous pouvez utiliser ce package pour sécuriser une installation de cluster en utilisant Apache Ranger et en opérant une intégration à Azure Active Directory. Pour plus d’informations, consultez [Vue d’ensemble de la sécurité d’entreprise dans Azure HDInsight](./domain-joined/hdinsight-security-overview.md).

Le package de sécurité d’entreprise vous permet d’intégrer HDInsight à Active Directory et Apache Ranger. Plusieurs utilisateurs peuvent être créés à l’aide du package de sécurité d’entreprise.

Pour plus d’informations sur la création d’un cluster HDInsight joint au domaine, consultez [Créer un environnement de bac à sable HDInsight joint à un domaine](./domain-joined/apache-domain-joined-configure-using-azure-adds.md).

### <a name="tls"></a>TLS

Pour plus d’informations, consultez [TLS](./transport-layer-security.md).

### <a name="virtual-network"></a>Réseau virtuel

Si votre solution nécessite des technologies qui sont réparties sur plusieurs types de clusters HDInsight, un [réseau virtuel Azure](../virtual-network/index.yml) peut connecter les types de cluster requis. Cette configuration permet aux clusters, et au code déployé sur ces clusters, de communiquer directement entre eux.

Pour plus d’informations sur l’utilisation d’un réseau virtuel Azure avec HDInsight, consultez [Planifier un réseau virtuel pour HDInsight](hdinsight-plan-virtual-network-deployment.md).

Pour voir un exemple d’utilisation de deux types de clusters au sein d’un réseau virtuel Azure, consultez [Utiliser Apache Spark Structured Streaming avec Kafka](hdinsight-apache-kafka-spark-structured-streaming.md). Pour plus d’informations sur l’utilisation de HDInsight avec un réseau virtuel, notamment la configuration spécifique requise pour le réseau virtuel, consultez [Planifier un réseau virtuel pour HDInsight](hdinsight-plan-virtual-network-deployment.md).

### <a name="disk-encryption-setting"></a>Paramètres de chiffrement des disques

Pour plus d’informations, consultez [Chiffrement de disque avec clé gérée par le client](./disk-encryption.md).

### <a name="kafka-rest-proxy"></a>Proxy REST Kafka

Ce paramètre est disponible uniquement pour le type de cluster Kafka. Pour plus d’informations, consultez [Utilisation d’un proxy REST](./kafka/rest-proxy.md).

### <a name="identity"></a>Identité

Pour plus d’informations, consultez [Identités managées dans Azure HDInsight](./hdinsight-managed-identities.md).

## <a name="configuration--pricing"></a>Configuration + prix

![Choix de la taille de votre nœud HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/azure-portal-cluster-configuration.png)

Vous êtes facturé pour l’utilisation du nœud tant que le cluster existe. La facturation démarre une fois le cluster créé et ne s’arrête que lorsque le cluster est supprimé. Les clusters ne peuvent pas être désalloués ou mis en attente.

### <a name="node-configuration"></a>Configuration de nœuds

Chaque type de cluster possède son propre nombre de nœuds, sa terminologie pour les nœuds et la taille de machine virtuelle par défaut. Dans le tableau suivant, les chiffres entre parenthèses correspondent au nombre de nœuds de chaque type.

| Type | Nœuds | Diagramme |
| --- | --- | --- |
| Hadoop |Nœud principal (2), nœud Worker (1+) |![Nœuds de cluster Hadoop HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-hadoop-cluster-type-nodes.png) |
| hbase |Serveur principal (2), serveur de région (1+), nœud principal/ZooKeeper (3) |![Configuration du type de cluster HDInsight HBase](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-hbase-cluster-type-setup.png) |
| Storm |Nœud Nimbus (2), serveur supervisor (1+), nœud ZooKeeper (3) |![Configuration du type de cluster Storm HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-storm-cluster-type-setup.png) |
| Spark |Nœud principal (2), nœud Worker (1+), nœud ZooKeeper (3) (gratuits pour les machines virtuelles ZooKeeper A1) |![Configuration du type de cluster Spark HDInsight](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-spark-cluster-type-setup.png) |

Pour plus d’informations, consultez [Configuration des nœuds par défaut et tailles des machines virtuelles pour les clusters](hdinsight-component-versioning.md#default-node-configuration-and-virtual-machine-sizes-for-clusters) dans « Quels sont les composants Hadoop et les versions dans HDInsight ? »

Le coût des clusters HDInsight est déterminé par le nombre de nœuds et par la taille des machines virtuelles pour les nœuds.

Les différents types de clusters ont des types de nœuds, nombres de nœuds et tailles de nœuds différents :
* Type de cluster Hadoop par défaut :
    * Deux *nœuds principaux*  
    * Quatre *nœuds Worker*
* Type de cluster Storm par défaut :
    * Deux *nœuds Nimbus*
    * Trois *nœuds ZooKeeper*
    * Quatre *nœuds superviseurs*

Si vous essayez simplement HDInsight, nous vous recommandons d’utiliser un nœud Worker. Pour plus d'informations sur la tarification de HDInsight, consultez la rubrique [Tarification HDInsight](https://go.microsoft.com/fwLink/?LinkID=282635&clcid=0x409).

> [!NOTE]  
> La limite de taille du cluster varie selon les abonnements Azure. Contactez le [support de facturation Azure](../azure-portal/supportability/how-to-create-azure-support-request.md) pour augmenter la limite.

Quand vous utilisez le portail Azure pour configurer le cluster, la taille de nœud est indiquée sous l’onglet **Configuration + prix** . Dans le portail, vous pouvez également voir le coût associé aux différentes tailles de nœuds.

### <a name="virtual-machine-sizes"></a>Tailles de machines virtuelles

Lorsque vous déployez des clusters, choisissez les ressources de calcul basées sur la solution que vous envisagez de déployer. Les machines virtuelles suivantes sont utilisées pour des clusters HDInsight :

* Machines virtuelles des séries A et D1-4 : [Tailles des machines virtuelles Linux à usage général](../virtual-machines/sizes-general.md)
* Machine virtuelle de la série D11-14 : [Tailles de machine virtuelle Linux optimisées pour la mémoire](../virtual-machines/sizes-memory.md)

Pour connaître la valeur que vous devez utiliser pour spécifier une taille de machine virtuelle lors de la création d’un cluster à l’aide de kits de développement logiciel différents ou lorsque vous utilisez Azure PowerShell, consultez les [tailles de machines virtuelles à utiliser pour les clusters HDInsight](../cloud-services/cloud-services-sizes-specs.md#size-tables). À partir de cet article lié, utilisez la valeur de la colonne **Taille** des tables.

> [!IMPORTANT]  
> Si vous avez besoin de plus de 32 nœuds Worker dans un cluster, vous devez sélectionner une taille de nœud principal avec au moins 8 cœurs et 14 Go de RAM.

Pour plus d’informations, consultez [Tailles des machines virtuelles](../virtual-machines/sizes.md). Pour plus d’informations sur la tarification des différentes tailles, consultez [Tarification de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight).

### <a name="add-application"></a>Ajouter l’application

Une application HDInsight est une application que les utilisateurs peuvent installer sur un cluster HDInsight sous Linux. Vous pouvez utiliser des applications fournies par Microsoft, des tiers ou que vous développez vous-même. Pour plus d’informations, consultez [Installer des applications Apache Hadoop tierces sur Azure HDInsight](hdinsight-apps-install-applications.md).

La plupart des applications HDInsight sont installées sur un nœud de périmètre vide.  Un nœud de périmètre vide est une machine virtuelle Linux avec les mêmes outils client installés et configurés comme dans le nœud principal. Vous pouvez utiliser le nœud de périmètre pour accéder au cluster, tester vos applications clientes et héberger vos applications clientes. Pour plus d’informations, consultez [Utiliser des nœuds de périmètre vides dans HDInsight](hdinsight-apps-use-edge-node.md).

### <a name="script-actions"></a>Actions de script

Vous pouvez installer des composants supplémentaires ou personnaliser la configuration de cluster à l’aide de scripts lors de la création. Ces scripts sont appelés à l’aide de l’option **action de script** , une option de configuration qui peut être utilisée à partir du portail Azure, de cmdlets HDInsight Windows PowerShell ou du Kit de développement logiciel (SDK) HDInsight .NET. Pour plus d’informations, consultez la page [Personnalisation d’un cluster HDInsight à l’aide d’une d’action de script](hdinsight-hadoop-customize-cluster-linux.md).

Certains composants Java natifs, comme Apache Mahout et Cascading, peuvent être exécutés sur le cluster en tant que fichiers Java Archive (JAR). Ces fichiers JAR peuvent être distribués au stockage Azure et envoyés aux clusters HDInsight à l’aide des mécanismes de soumission des travaux Hadoop. Pour plus d’informations, consultez [Envoi de tâches Apache Hadoop par programmation](hadoop/submit-apache-hadoop-jobs-programmatically.md).

> [!NOTE]  
> En cas de problèmes lors du déploiement ou de l’appel des fichiers JAR sur les clusters HDInsight, contactez le [support Microsoft](https://azure.microsoft.com/support/options/).
>
> Cascading n’est pas pris en charge par HDInsight et ne peut pas bénéficier du support Microsoft. Pour obtenir la liste des composants pris en charge, consultez la rubrique [Nouveautés des versions de cluster fournies par HDInsight](hdinsight-component-versioning.md).

Vous souhaitez parfois configurer des fichiers de configuration suivants pendant le processus de création :

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

Pour plus d’informations, consultez [Personnalisation de clusters HDInsight à l’aide de Bootstrap](hdinsight-hadoop-customize-cluster-bootstrap.md).

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre les problèmes de création de cluster avec Azure HDInsight](./hadoop/hdinsight-troubleshoot-cluster-creation-fails.md)
* [Présentation d’HDInsight, de l’écosystème Apache Hadoop et des clusters Hadoop](hadoop/apache-hadoop-introduction.md)
* [Bien démarrer avec Apache Hadoop dans HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Travailler à partir d’un PC Windows dans Apache Hadoop sur HDInsight](hdinsight-hadoop-windows-tools.md)