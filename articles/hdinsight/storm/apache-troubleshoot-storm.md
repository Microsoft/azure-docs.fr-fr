---
title: Résolution de problèmes Storm à l’aide d’Azure HDInsight
description: Obtenez les réponses aux questions courantes sur l’utilisation d’Apache Storm avec Azure HDInsight.
keywords: Azure HDInsight, Storm, FAQ, guide de dépannage, problèmes courants
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 11/08/2019
ms.custom: seodec18
ms.openlocfilehash: fc08adb2f71c49fad3c8574e9cfd5315f13d7f44
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95982260"
---
# <a name="troubleshoot-apache-storm-by-using-azure-hdinsight"></a>Résolution de problèmes Apache Storm à l’aide d’Azure HDInsight

Découvrez les principaux problèmes rencontrés lors de l’utilisation de charges utiles [Apache Storm](https://storm.apache.org/) dans [Apache Ambari](https://ambari.apache.org/), et leur résolution.

## <a name="how-do-i-access-the-storm-ui-on-a-cluster"></a>Comment accéder à l’interface utilisateur Storm sur un cluster ?

Deux options vous permettent d’accéder à l’interface utilisateur Storm à partir d’un navigateur :

### <a name="apache-ambari-ui"></a>Interface utilisateur Apache Ambari

1. Accédez au tableau de bord Ambari.
2. Dans la liste des services, sélectionnez **Storm**.
3. Dans le menu **Quick Links** (Liens rapides), sélectionnez **Storm UI**.

### <a name="direct-link"></a>Lien direct

Vous pouvez accéder à l’interface utilisateur de Storm à l’URL suivante :

`https://<cluster DNS name>/stormui`

Exemple : `https://stormcluster.azurehdinsight.net/stormui`

## <a name="how-do-i-transfer-storm-event-hub-spout-checkpoint-information-from-one-topology-to-another"></a>Comment transférer les informations de point de contrôle du spout de concentrateurs d’événements Storm depuis une topologie vers une autre ?

Lorsque vous développez des topologies qui lisent à partir des hubs d’événements Azure en utilisant le fichier .jar du spout Storm Eventhub, vous devez déployer une topologie portant le même nom sur un nouveau cluster. Toutefois, vous devez conserver sur l’ancien cluster les données de point de contrôle qui ont été validées dans [Apache ZooKeeper](https://zookeeper.apache.org/).

### <a name="where-checkpoint-data-is-stored"></a>Où sont stockées les données de point de contrôle ?

Les données de point de contrôle des décalages sont enregistrées par le spout EventHub dans Zookeeper sous deux chemins d’accès racine :

- Les points de contrôle de Spout non transactionnels sont stockés dans `/eventhubspout`.

- Les données de points de contrôle de Spout transactionnels sont stockées dans `/transactional`.

### <a name="how-to-restore"></a>Comment effectuer une restauration ?

Pour obtenir les scripts et les bibliothèques utilisés pour exporter des données depuis ZooKeeper et les réimporter sous un nom différent, consultez les [exemples HDInsight Storm](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/tools/zkdatatool-1.0).

Le dossier lib renferme des fichiers .jar contenant les instructions d’implémentation de l’opération d’exportation/importation. Le dossier bash comporte un exemple de script montrant comment exporter des données depuis le serveur ZooKeeper de l’ancien cluster et les réimporter dans le serveur ZooKeeper du nouveau cluster.

Exécutez le script [stormmeta.sh](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/tools/zkdatatool-1.0/bash/stormmeta.sh) à partir des nœuds ZooKeeper pour exporter et réimporter des données. Indiquez la version appropriée de Hortonworks Data Platform (HDP) dans le script. (Nous travaillons sur la création de scripts génériques dans HDInsight. Les scripts génériques peuvent s’exécuter à partir de n’importe quel nœud du cluster sans modifications de l’utilisateur.)

La commande d’exportation écrit les métadonnées sur un chemin Apache HDFS (Hadoop Distributed File System) (dans Stockage Blob Azure ou Azure Data Lake Storage) à un emplacement que vous définissez.

### <a name="examples"></a>Exemples

#### <a name="export-offset-metadata"></a>Exporter les métadonnées de décalage

1. À l’aide de SSH, accédez à l’ancien cluster ZooKeeper à partir duquel les données de décalage de point de contrôle doivent être exportées.
2. Exécutez la commande suivante (après avoir mis à jour la chaîne de la version HDP) pour exporter les données de décalage ZooKeeper vers le chemin de stockage HDFS `/stormmetadta/zkdata` :

    ```apache
    java -cp ./*:/etc/hadoop/conf/*:/usr/hdp/2.5.1.0-56/hadoop/*:/usr/hdp/2.5.1.0-56/hadoop/lib/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/lib/*:/etc/failover-controller/conf/*:/etc/hadoop/* com.microsoft.storm.zkdatatool.ZkdataImporter export /eventhubspout /stormmetadata/zkdata
    ```

#### <a name="import-offset-metadata"></a>Importer les métadonnées de décalage

1. À l’aide de SSH, accédez à l’ancien cluster ZooKeeper à partir duquel les données de décalage de point de contrôle doivent être importées.
2. Exécutez la commande suivante (après avoir mis à jour la chaîne de la version HDP) pour importer les données de décalage ZooKeeper à partir du chemin de stockage HDFS `/stormmetadata/zkdata` vers le serveur ZooKeeper dans le cluster cible :

    ```apache
    java -cp ./*:/etc/hadoop/conf/*:/usr/hdp/2.5.1.0-56/hadoop/*:/usr/hdp/2.5.1.0-56/hadoop/lib/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/lib/*:/etc/failover-controller/conf/*:/etc/hadoop/* com.microsoft.storm.zkdatatool.ZkdataImporter import /eventhubspout /home/sshadmin/zkdata
    ```

#### <a name="delete-offset-metadata-so-that-topologies-can-start-processing-data-from-the-beginning-or-from-a-timestamp-that-the-user-chooses"></a>Supprimer les métadonnées de décalage afin que les topologies puissent commencer à traiter les données à partir du début ou d’un horodatage défini par l’utilisateur

1. À l’aide de SSH, accédez à l’ancien cluster ZooKeeper à partir duquel les données de décalage de point de contrôle doivent être supprimées.
2. Exécutez la commande suivante (après avoir mis à jour la chaîne de la version HDP) pour supprimer les données de décalage ZooKeeper du cluster actuel :

    ```apache
    java -cp ./*:/etc/hadoop/conf/*:/usr/hdp/2.5.1.0-56/hadoop/*:/usr/hdp/2.5.1.0-56/hadoop/lib/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/*:/usr/hdp/2.5.1.0-56/hadoop-hdfs/lib/*:/etc/failover-controller/conf/*:/etc/hadoop/* com.microsoft.storm.zkdatatool.ZkdataImporter delete /eventhubspout
    ```

## <a name="how-do-i-locate-storm-binaries-on-a-cluster"></a>Comment localiser les fichiers binaires de Storm sur un cluster ?

Les fichiers binaires Storm de la pile HDP actuelle se trouvent à l’emplacement `/usr/hdp/current/storm-client`. L’emplacement est le même pour les nœuds principaux et les nœuds de travail.

Il peut y avoir plusieurs fichiers binaires pour des versions spécifiques de HDP à l’emplacement /usr/hdp (par exemple, `/usr/hdp/2.5.0.1233/storm`). Un lien symbolique avec le dossier `/usr/hdp/current/storm-client` est créé vers la version la plus récente exécutée sur le cluster.

Pour plus d’informations, consultez [Se connecter à HDInsight (Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) et [Apache Storm](https://storm.apache.org/).

## <a name="how-do-i-determine-the-deployment-topology-of-a-storm-cluster"></a>Comment déterminer la topologie de déploiement d’un cluster Storm ?

Commencez par identifier tous les composants installés avec HDInsight Storm. Un cluster Storm se compose de quatre catégories de nœuds :

* Nœuds de passerelle
* Nœuds principaux
* Nœuds ZooKeeper
* Nœuds de travail

### <a name="gateway-nodes"></a>Nœuds de passerelle

Un nœud de passerelle est un service de passerelle et de proxy inverse qui permet un accès public à un service de gestion Ambari actif. Il gère également le choix du leader Ambari.

### <a name="head-nodes"></a>Nœuds principaux

Les nœuds principaux Storm exécutent les services suivants :
* Nimbus
* Ambari Server
* Ambari Metrics Server
* Ambari Metrics Collector
 
### <a name="zookeeper-nodes"></a>Nœuds ZooKeeper

HDInsight est fourni avec un quorum de trois nœuds ZooKeeper. Sa taille est fixe et n’est pas reconfigurable.

Les services Storm du cluster sont configurés pour utiliser le quorum ZooKeeper automatiquement.

### <a name="worker-nodes"></a>Nœuds de travail

Les nœuds de travail Storm exécutent les services suivants :
* Supervisor
* Machines virtuelles Java (JVM) de travail, pour l’exécution des topologies
* Ambari Agent

## <a name="how-do-i-locate-storm-event-hub-spout-binaries-for-development"></a>Comment localiser les fichiers binaires du spout de concentrateurs d’événements Storm pour le développement ?

Pour plus d’informations sur l’utilisation de fichiers .jar de spout EventHub Storm avec votre topologie, consultez les ressources suivantes.

### <a name="java-based-topology"></a>Topologie basée sur Java

[Traitement des événements Azure Event Hubs avec Apache Storm sur HDInsight (Java)](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub)

### <a name="c-based-topology-mono-on-hdinsight-34-linux-storm-clusters"></a>Topologie basée sur C# (Mono sur les clusters Storm HDInsight 3.4+ Linux)

[Traitement des événements Azure Event Hubs avec Apache Storm sur HDInsight (C#)](./apache-storm-develop-csharp-event-hub-topology.md)

### <a name="latest-apache-storm-event-hub-spout-binaries-for-hdinsight-35-linux-storm-clusters"></a>Fichiers binaires de spout EventHub Apache Storm les plus récents pour les clusters Storm HDInsight 3.5+ Linux

Pour savoir comment utiliser le Spout EventHub Storm le plus récent compatible avec les clusters Storm HDInsight 3.5+ Linux, consultez le [fichier readme du dépôt mvn-repo](https://github.com/hdinsight/mvn-repo/blob/master/README.md).

### <a name="source-code-examples"></a>Exemples de code source

Consultez ces [exemples](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub) pour savoir comment lire et écrire à partir d’Azure EventHub à l’aide d’une topologie Apache Storm (écrite en Java) sur un cluster Azure HDInsight.

## <a name="how-do-i-locate-storm-log4j-2-configuration-files-on-clusters"></a>Comment localiser les fichiers de configuration Storm Log4J 2 sur les clusters ?

Pour identifier les fichiers de configuration [Apache Log4j 2](https://logging.apache.org/log4j/2.x/) des services Storm.

### <a name="on-head-nodes"></a>Sur les nœuds principaux

La configuration Log4J de Nimbus est lue à partir de `/usr/hdp/\<HDP version>/storm/log4j2/cluster.xml`.

### <a name="on-worker-nodes"></a>Sur les nœuds de travail

La configuration Log4J du superviseur est lue à partir de `/usr/hdp/\<HDP version>/storm/log4j2/cluster.xml`.

Le fichier de configuration Log4J du Worker est lu à partir de `/usr/hdp/\<HDP version>/storm/log4j2/worker.xml`.

Exemples : `/usr/hdp/2.6.0.2-76/storm/log4j2/cluster.xml`
`/usr/hdp/2.6.0.2-76/storm/log4j2/worker.xml`

---

## <a name="not-a-leader-exception"></a>Exception non leader

Lors de l’envoi d’une topologie de réseau, l’utilisateur peut recevoir un message d’erreur semblable à celui-ci : `Topology submission exception, cause not a leader, the current leader is NimbusInfo`.

Pour le résoudre, l’utilisateur devra peut-être créer un ticket pour faire redémarrer les nœuds. Pour plus d’informations, consultez [https://community.hortonworks.com/content/supportkb/150287/error-ignoring-exception-while-trying-to-get-leade.html](https://community.hortonworks.com/content/supportkb/150287/error-ignoring-exception-while-trying-to-get-leade.html).

---

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

- Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

- Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

- Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support**. Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).