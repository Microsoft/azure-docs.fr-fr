---
title: Accéder aux journaux des applications Apache Hadoop YARN - Azure HDInsight
description: Découvrez comment accéder aux journaux des applications YARN sur un cluster HDInsight sous Linux (Apache Hadoop) à l’aide de la ligne de commande et d’un navigateur web.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: 726cf362e62f0ef914dfaea090a08c224bd5d8d6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001947"
---
# <a name="access-apache-hadoop-yarn-application-logs-on-linux-based-hdinsight"></a>Accéder aux journaux des applications Apache Hadoop YARN dans HDInsight basé sur Linux

Découvrez comment accéder aux journaux des applications [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) (Yet Another Resource Negotiator) sur un cluster Apache Hadoop dans Azure HDInsight.

## <a name="what-is-apache-yarn"></a>Qu’est-ce qu’Apache YARN ?

YARN (Yet Another Resource Negotiator) prend en charge plusieurs modèles de programmation (dont Apache Hadoop MapReduce) en séparant la gestion des ressources de la planification et de la supervision des applications. YARN utilise les éléments suivants : *`ResourceManager`* (RM) global, *NodeManagers* (NM) par nœud worker et *ApplicationMasters* (AM) par application. Le maître d'application, propre à chaque application, négocie les ressources nécessaires (processeur, mémoire, disque, réseau) pour exécuter l'application avec le gestionnaire de ressources. Le gestionnaire de ressources fonctionne avec les gestionnaires de nœuds pour octroyer ces ressources sous forme de *conteneurs*. Le maître d'application est chargé de suivre la progression des conteneurs qui lui sont assignés par le gestionnaire de ressources. Selon la nature de l'application, celle-ci peut nécessiter de nombreux conteneurs.

Chaque application peut comporter plusieurs *tentatives d’application*. Si une application échoue, vous pouvez la relancer. Il s’agit alors d’une nouvelle tentative. Chaque tentative est exécutée dans un conteneur. D’une certaine manière, un conteneur fournit le contexte pour l’unité de base du travail effectué par une application YARN. Tout le travail réalisé dans le contexte d’un conteneur est effectué sur l’unique nœud Worker auquel le conteneur a été donné. Pour plus d’informations, consultez [Hadoop : Écriture d’applications YARN](https://hadoop.apache.org/docs/r2.7.4/hadoop-yarn/hadoop-yarn-site/WritingYarnApplications.html) ou [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html).

Pour mettre à l’échelle votre cluster de manière à prendre en charge un plus grand débit de traitement, vous pouvez utiliser [Mettre à l’échelle automatiquement](hdinsight-autoscale-clusters.md) ou [Mettre à l’échelle vos clusters manuellement à l’aide de différents langages](hdinsight-scaling-best-practices.md#utilities-to-scale-clusters).

## <a name="yarn-timeline-server"></a>YARN Timeline Server

[Apache Hadoop YARN Timeline Server](https://hadoop.apache.org/docs/r2.7.3/hadoop-yarn/hadoop-yarn-site/TimelineServer.html) fournit des informations génériques sur les applications terminées

YARN Timeline Server inclut le type de données suivant :

* ID d’application, identificateur unique d’une application
* Utilisateur ayant démarré l’application
* Informations sur les tentatives effectuées afin de terminer l’application
* Conteneurs utilisés par toute tentative d’application donnée

## <a name="yarn-applications-and-logs"></a>Applications et journaux d’activité YARN

Les journaux des applications (et les journaux d’activité des conteneurs associés) sont essentiels pour déboguer des applications Hadoop problématiques. YARN fournit une infrastructure adaptée à la collecte, à l’agrégation et au stockage des journaux des applications avec l’[agrégation des journaux](https://hortonworks.com/blog/simplifying-user-logs-management-and-access-in-yarn/).

La fonction d’agrégation de journaux d’activité rend l’accès aux journaux des applications plus déterministe. Il regroupe les journaux d’activité de tous les conteneurs sur un nœud Worker et les stocke dans un fichier journal agrégé par nœud Worker. Le journal est stocké sur le système de fichiers par défaut après la fin d’une application. Votre application peut utiliser des centaines voire des milliers de conteneurs, mais les journaux d’activité de tous les conteneurs exécutés sur un nœud worker unique sont toujours regroupés dans un fichier unique. Cela se traduit par l’existence d’un seul fichier journal par nœud worker utilisé par votre application. L’agrégation de journaux est activée par défaut sur les clusters HDInsight version 3.0 et versions ultérieures. Les journaux d’activité agrégés sont situés dans le stockage par défaut pour le cluster. Le chemin d’accès suivant est le chemin d’accès HDFS pour les journaux d’activité :

```
/app-logs/<user>/logs/<applicationId>
```

Dans le chemin d’accès, `user` est le nom de l’utilisateur qui a démarré l’application. Le `applicationId` est l’identificateur unique affecté à une application par le Gestionnaire de ressources YARN.

Les journaux agrégés ne sont pas lisibles directement, car ils sont écrits dans un TFile, format binaire indexé par conteneur. Utilisez les journaux `ResourceManager` YARN ou les outils CLI pour afficher ces journaux au format texte brut pour les applications ou les conteneurs qui présentent un intérêt.

## <a name="yarn-logs-in-an-esp-cluster"></a>Journaux Yarn dans un cluster ESP

Deux configurations doivent être ajoutées à l'élément `mapred-site` personnalisé dans Ambari.

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`, où `CLUSTERNAME` est le nom de votre cluster.

1. À partir de l’interface utilisateur Ambari, accédez à **MapReduce2** > **Configurations** > **Avancé** > **mapred-site personnalisé**.

1. Ajoutez *un* des ensembles de propriétés suivants :

    **Ensemble 1**

    ```
    mapred.acls.enabled=true
    mapreduce.job.acl-view-job=*
    ```

    **Ensemble 2**

    ```
    mapreduce.job.acl-view-job=<user1>,<user2>,<user3>
    ```

1. Enregistrez les modifications, puis redémarrez tous les services affectés.

## <a name="yarn-cli-tools"></a>Outils de l’interface de ligne de commande YARN

1. Utilisez la [commande ssh](./hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Dressez la liste des ID de toutes les applications Yarn en cours d’exécution à l’aide de la commande suivante :

    ```bash
    yarn top
    ```

    Notez l’ID d’application de la colonne `APPLICATIONID` dont les journaux doivent être téléchargés.

    ```output
    YARN top - 18:00:07, up 19d, 0:14, 0 active users, queue(s): root
    NodeManager(s): 4 total, 4 active, 0 unhealthy, 0 decommissioned, 0 lost, 0 rebooted
    Queue(s) Applications: 2 running, 10 submitted, 0 pending, 8 completed, 0 killed, 0 failed
    Queue(s) Mem(GB): 97 available, 3 allocated, 0 pending, 0 reserved
    Queue(s) VCores: 58 available, 2 allocated, 0 pending, 0 reserved
    Queue(s) Containers: 2 allocated, 0 pending, 0 reserved
    
                      APPLICATIONID USER             TYPE      QUEUE   #CONT  #RCONT  VCORES RVCORES     MEM    RMEM  VCORESECS    MEMSECS %PROGR       TIME NAME
     application_1490377567345_0007 hive            spark  thriftsvr       1       0       1       0      1G      0G    1628407    2442611  10.00   18:20:20 Thrift JDBC/ODBC Server
     application_1490377567345_0006 hive            spark  thriftsvr       1       0       1       0      1G      0G    1628430    2442645  10.00   18:20:20 Thrift JDBC/ODBC Server
    ```

1. Vous pouvez afficher ces journaux d’activité en texte brut en exécutant l’une des commandes suivantes :

    ```bash
    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application>
    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application> -containerId <containerId> -nodeAddress <worker-node-address>
    ```

    Définissez les valeurs &lt;applicationId>, &lt;user-who-started-the-application>, &lt;containerId>, et &lt;worker-node-address> lorsque vous exécutez ces commandes.

### <a name="other-sample-commands"></a>Autres exemples de commandes

1. Télécharger les journaux d’activité de conteneurs Yarn pour tous les processus maîtres d’application à l’aide de la commande ci-dessous. Cette étape crée le fichier journal nommé `amlogs.txt` au format texte.

    ```bash
    yarn logs -applicationId <application_id> -am ALL > amlogs.txt
    ```

1. Téléchargez les journaux d’activité de conteneurs Yarn uniquement pour les processus maîtres d’application les plus récents à l’aide de la commande suivante :

    ```bash
    yarn logs -applicationId <application_id> -am -1 > latestamlogs.txt
    ```

1. Télécharger les journaux d’activité de conteneurs YARN pour les deux premiers processus maîtres d’application à l’aide de la commande suivante :

    ```bash
    yarn logs -applicationId <application_id> -am 1,2 > first2amlogs.txt
    ```

1. Téléchargez tous les journaux d’activité de conteneurs YARN à l’aide de la commande suivante :

    ```bash
    yarn logs -applicationId <application_id> > logs.txt
    ```

1. Téléchargez le journal de conteneur Yarn pour un conteneur particulier à l’aide de la commande suivante :

    ```bash
    yarn logs -applicationId <application_id> -containerId <container_id> > containerlogs.txt
    ```

## <a name="yarn-resourcemanager-ui"></a>Interface utilisateur `ResourceManager` YARN

L’interface utilisateur `ResourceManager` YARN s’exécute sur le nœud principal du cluster. Il est accessible via l’interface utilisateur web d’Ambari. Procédez comme suit pour afficher les journaux d’activité YARN :

1. Dans votre navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`. Remplacez CLUSTERNAME par le nom de votre cluster HDInsight.

2. Dans la liste des services sur la gauche, sélectionnez **YARN**.

    ![Apache Ambari - Service Yarn sélectionné](./media/hdinsight-hadoop-access-yarn-app-logs-linux/yarn-service-selected.png)

3. Dans la liste déroulante **Liens rapides**, sélectionnez un des nœuds principaux du cluster, puis **`ResourceManager Log`** .

    ![Apache Ambari - Liens rapides Yarn](./media/hdinsight-hadoop-access-yarn-app-logs-linux/hdi-yarn-quick-links.png)

    Une liste de liens menant vers les journaux d’activité YARN s’affiche.

## <a name="next-steps"></a>Étapes suivantes

* [Architecture Apache Hadoop dans HDInsight](hdinsight-hadoop-architecture.md)
* [Résoudre les problèmes d’Apache Hadoop YARN avec Azure HDInsight](hdinsight-troubleshoot-yarn.md)
