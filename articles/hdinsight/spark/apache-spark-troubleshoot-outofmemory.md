---
title: Exceptions OutOfMemoryError pour Apache Spark dans Azure HDInsight
description: Différentes exceptions OutOfMemoryError pour cluster Apache Spark dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/15/2019
ms.openlocfilehash: 156d3ce4b7f8b6eaa297f2cddd0d5a93f382f78e
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547280"
---
# <a name="outofmemoryerror-exceptions-for-apache-spark-in-azure-hdinsight"></a>Exceptions OutOfMemoryError pour Apache Spark dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Apache Spark dans des clusters Azure HDInsight.

## <a name="scenario-outofmemoryerror-exception-for-apache-spark"></a>Scénario : Exception OutOfMemoryError pour Apache Spark

### <a name="issue"></a>Problème

Échec de l'application Apache Spark avec une exception non prise en charge OutOfMemoryError. Vous pouvez recevoir un message d’erreur similaire au suivant :

```error
ERROR Executor: Exception in task 7.0 in stage 6.0 (TID 439)

java.lang.OutOfMemoryError
    at java.io.ByteArrayOutputStream.hugeCapacity(Unknown Source)
    at java.io.ByteArrayOutputStream.grow(Unknown Source)
    at java.io.ByteArrayOutputStream.ensureCapacity(Unknown Source)
    at java.io.ByteArrayOutputStream.write(Unknown Source)
    at java.io.ObjectOutputStream$BlockDataOutputStream.drain(Unknown Source)
    at java.io.ObjectOutputStream$BlockDataOutputStream.setBlockDataMode(Unknown Source)
    at java.io.ObjectOutputStream.writeObject0(Unknown Source)
    at java.io.ObjectOutputStream.writeObject(Unknown Source)
    at org.apache.spark.serializer.JavaSerializationStream.writeObject(JavaSerializer.scala:44)
    at org.apache.spark.serializer.JavaSerializerInstance.serialize(JavaSerializer.scala:101)
    at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:239)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
    at java.lang.Thread.run(Unknown Source)
```

```error
ERROR SparkUncaughtExceptionHandler: Uncaught exception in thread Thread[Executor task launch worker-0,5,main]

java.lang.OutOfMemoryError
    at java.io.ByteArrayOutputStream.hugeCapacity(Unknown Source)
    ...
```

### <a name="cause"></a>Cause

La cause la plus probable de cette exception est que le segment de mémoire allouée aux machines virtuelles Java (JVM) est insuffisant. Ces JVM sont lancées en tant qu’exécuteurs ou pilotes dans le cadre de l’application Apache Spark.

### <a name="resolution"></a>Résolution

1. Déterminez le volume maximal de données que l’application Spark va traiter. Estimez ce volume en fonction du volume maximal des données d’entrée, des données intermédiaires produites par la transformation des données d’entrée et des données de sortie produites à partir des données intermédiaires. Si l’estimation initiale n’est pas suffisante, augmentez légèrement le volume, et itérez jusqu'à ce que les erreurs de mémoire diminuent.

1. Assurez-vous que le cluster HDInsight à utiliser dispose de suffisamment de ressources en termes de mémoire et de cœurs pour prendre en charge l’application Spark. Pour le vérifier, accédez à la section Mesures de cluster de l’interface utilisateur YARN du cluster et comparez les valeurs **Mémoire utilisée** / **Mémoire totale** et les valeurs **VCores utilisées** / **Total des VCores** .

    ![affichage de la mémoire centrale yarn](./media/apache-spark-ts-outofmemory/yarn-core-memory-view.png)

1. Définissez les configurations Spark suivantes sur les valeurs qui conviennent. Équilibrez les exigences de l’application avec les ressources disponibles dans le cluster. Ces valeurs ne doivent pas dépasser 90 % de la mémoire et des cœurs disponibles, comme affiché par YARN, et doivent également respecter la mémoire minimale requise de l’application Spark :

    ```
    spark.executor.instances (Example: 8 for 8 executor count)
    spark.executor.memory (Example: 4g for 4 GB)
    spark.yarn.executor.memoryOverhead (Example: 384m for 384 MB)
    spark.executor.cores (Example: 2 for 2 cores per executor)
    spark.driver.memory (Example: 8g for 8GB)
    spark.driver.cores (Example: 4 for 4 cores)
    spark.yarn.driver.memoryOverhead (Example: 384m for 384MB)
    ```

    Mémoire totale utilisée par tous les exécuteurs =

    ```
    spark.executor.instances * (spark.executor.memory + spark.yarn.executor.memoryOverhead) 
    ```

    Mémoire totale utilisée par le pilote =

    ```
    spark.driver.memory + spark.yarn.driver.memoryOverhead
    ```

---

## <a name="scenario-java-heap-space-error-when-trying-to-open-apache-spark-history-server"></a>Scénario : Erreur d’espace du tas Java lors de la tentative d’ouverture du serveur d’historique Apache Spark

### <a name="issue"></a>Problème

Vous recevez l’erreur suivante lors de l’ouverture d’événements dans le serveur d’historique Spark :

```
scala.MatchError: java.lang.OutOfMemoryError: Java heap space (of class java.lang.OutOfMemoryError)
```

### <a name="cause"></a>Cause

Ce problème est souvent dû à un manque de ressources lors de l’ouverture de fichiers Spark Event volumineux. La taille du tas Spark est définie sur 1 Go par défaut, mais les fichiers d’événements Spark volumineux peuvent nécessiter une taille supérieure.

Pour vérifier la taille des fichiers que vous essayez de charger, vous pouvez exécuter les commandes suivantes :

```bash
hadoop fs -du -s -h wasb:///hdp/spark2-events/application_1503957839788_0274_1/
**576.5 M**  wasb:///hdp/spark2-events/application_1503957839788_0274_1

hadoop fs -du -s -h wasb:///hdp/spark2-events/application_1503957839788_0264_1/
**2.1 G**  wasb:///hdp/spark2-events/application_1503957839788_0264_1
```

### <a name="resolution"></a>Résolution

Vous pouvez augmenter la mémoire du serveur d’historique Spark en modifiant la propriété `SPARK_DAEMON_MEMORY` dans la configuration Spark et en redémarrant tous les services.

Pour cela, accédez à l’interface utilisateur du navigateur Ambari, puis sélectionnez la section Spark2/Config/Advanced spark2-env.

![Section Advanced spark2-env](./media/apache-spark-ts-outofmemory-heap-space/apache-spark-image01.png)

Ajoutez la propriété suivante pour que la mémoire du serveur d’historique Spark passe de 1 à 4 Go : `SPARK_DAEMON_MEMORY=4g`.

![Propriété Spark](./media/apache-spark-ts-outofmemory-heap-space/apache-spark-image02.png)

Veillez à redémarrer tous les services affectés à partir d’Ambari.

---

## <a name="scenario-livy-server-fails-to-start-on-apache-spark-cluster"></a>Scénario : Échec du démarrage du serveur Livy sur le cluster Apache Spark

### <a name="issue"></a>Problème

Impossible de démarrer Livy Server sur un Apache Spark [(Spark 2.1 sur Linux (HDI 3.6)]. Toute tentative de redémarrage entraîne la pile d’erreurs suivante, provenant des journaux Livy :

```log
17/07/27 17:52:50 INFO CuratorFrameworkImpl: Starting
17/07/27 17:52:50 INFO ZooKeeper: Client environment:zookeeper.version=3.4.6-29--1, built on 05/15/2017 17:55 GMT
17/07/27 17:52:50 INFO ZooKeeper: Client environment:host.name=10.0.0.66
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.version=1.8.0_131
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.vendor=Oracle Corporation
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.home=/usr/lib/jvm/java-8-openjdk-amd64/jre
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.class.path= <DELETED>
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.library.path= <DELETED>
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.io.tmpdir=/tmp
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.compiler=<NA>
17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.name=Linux
17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.arch=amd64
17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.version=4.4.0-81-generic
17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.name=livy
17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.home=/home/livy
17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.dir=/home/livy
17/07/27 17:52:50 INFO ZooKeeper: Initiating client connection, connectString=zk2-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181,zk3-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181,zk6-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181 sessionTimeout=60000 watcher=org.apache.curator.ConnectionState@25fb8912
17/07/27 17:52:50 INFO StateStore$: Using ZooKeeperStateStore for recovery.
17/07/27 17:52:50 INFO ClientCnxn: Opening socket connection to server 10.0.0.61/10.0.0.61:2181. Will not attempt to authenticate using SASL (unknown error)
17/07/27 17:52:50 INFO ClientCnxn: Socket connection established to 10.0.0.61/10.0.0.61:2181, initiating session
17/07/27 17:52:50 INFO ClientCnxn: Session establishment complete on server 10.0.0.61/10.0.0.61:2181, sessionid = 0x25d666f311d00b3, negotiated timeout = 60000
17/07/27 17:52:50 INFO ConnectionStateManager: State change: CONNECTED
17/07/27 17:52:50 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
17/07/27 17:52:50 INFO AHSProxy: Connecting to Application History server at headnodehost/10.0.0.67:10200
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
  at java.lang.Thread.start0(Native Method)
  at java.lang.Thread.start(Thread.java:717)
  at com.cloudera.livy.Utils$.startDaemonThread(Utils.scala:98)
  at com.cloudera.livy.utils.SparkYarnApp.<init>(SparkYarnApp.scala:232)
  at com.cloudera.livy.utils.SparkApp$.create(SparkApp.scala:93)
  at com.cloudera.livy.server.batch.BatchSession$$anonfun$recover$2$$anonfun$apply$4.apply(BatchSession.scala:117)
  at com.cloudera.livy.server.batch.BatchSession$$anonfun$recover$2$$anonfun$apply$4.apply(BatchSession.scala:116)
  at com.cloudera.livy.server.batch.BatchSession.<init>(BatchSession.scala:137)
  at com.cloudera.livy.server.batch.BatchSession$.recover(BatchSession.scala:108)
  at com.cloudera.livy.sessions.BatchSessionManager$$anonfun$$init$$1.apply(SessionManager.scala:47)
  at com.cloudera.livy.sessions.BatchSessionManager$$anonfun$$init$$1.apply(SessionManager.scala:47)
  at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:244)
  at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:244)
  at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
  at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:47)
  at scala.collection.TraversableLike$class.map(TraversableLike.scala:244)
  at scala.collection.AbstractTraversable.map(Traversable.scala:105)
  at com.cloudera.livy.sessions.SessionManager.com$cloudera$livy$sessions$SessionManager$$recover(SessionManager.scala:150)
  at com.cloudera.livy.sessions.SessionManager$$anonfun$1.apply(SessionManager.scala:82)
  at com.cloudera.livy.sessions.SessionManager$$anonfun$1.apply(SessionManager.scala:82)
  at scala.Option.getOrElse(Option.scala:120)
  at com.cloudera.livy.sessions.SessionManager.<init>(SessionManager.scala:82)
  at com.cloudera.livy.sessions.BatchSessionManager.<init>(SessionManager.scala:42)
  at com.cloudera.livy.server.LivyServer.start(LivyServer.scala:99)
  at com.cloudera.livy.server.LivyServer$.main(LivyServer.scala:302)
  at com.cloudera.livy.server.LivyServer.main(LivyServer.scala)
  
  ## using "vmstat" found  we had enough free memory
```

### <a name="cause"></a>Cause

`java.lang.OutOfMemoryError: unable to create new native thread` indique que le système d’exploitation ne peut pas attribuer plus de threads natifs aux JVM. Il est confirmé que cette exception est due à la violation de la limite du nombre de threads par processus.

Lorsque Livy Server se termine de manière inattendue, toutes les connexions aux clusters Spark sont également terminées, ce qui signifie que tous les travaux et les données associées sont perdues. Dans HDP 2.6, le mécanisme de récupération de session a été introduit, et Livy stocke les détails de session dans Zookeeper pour qu’ils puissent être récupérés après la restauration du serveur Livy.

Quand un grand nombre de travaux sont envoyés via Livy, dans le cadre de la haute disponibilité, Livy Server stocke ces états de session dans ZK (sur les clusters HDInsight) et récupère ces sessions lorsque le service Livy est redémarré. Lors du redémarrage après un arrêt inattendu, Livy crée un thread par session, ce qui accumule un certain nombre de sessions à récupérer en raison de la création d’un trop grand nombre de threads.

### <a name="resolution"></a>Résolution

Supprimez toutes les entrées en suivant les étapes décrites ci-dessous.

1. Procurez-vous l’adresse IP des nœuds Zookeeper en utilisant

    ```bash
    grep -R zk /etc/hadoop/conf  
    ```

1. La commande ci-dessus répertorie tous les Zookeepers pour mon cluster

    ```bash
    /etc/hadoop/conf/core-site.xml:      <value>zk1-hwxspa.lnuwp5akw5ie1j2gi2amtuuimc.dx.internal.cloudapp.net:2181,zk2-      hwxspa.lnuwp5akw5ie1j2gi2amtuuimc.dx.internal.cloudapp.net:2181,zk4-hwxspa.lnuwp5akw5ie1j2gi2amtuuimc.dx.internal.cloudapp.net:2181</value>
    ```

1. Récupérez toutes les adresses IP des nœuds Zookeeper à l’aide de la commande ping ou vous pouvez également vous connecter à Zookeeper à partir du nœud principal à l’aide du nom ZK

    ```bash
    /usr/hdp/current/zookeeper-client/bin/zkCli.sh -server zk2-hwxspa:2181
    ```

1. Une fois que vous êtes connecté à Zookeeper, exécutez la commande suivante pour répertorier toutes les sessions qui ont tenté de redémarrer.

    1. Dans la plupart des cas, il peut s’agir d’une liste de plus de 8 000 sessions ####

        ```bash
        ls /livy/v1/batch
        ```

    1. La commande suivante permet de supprimer toutes les sessions à récupérer. #####

        ```bash
        rmr /livy/v1/batch
        ```

1. Attendez la fin de la commande ci-dessus et le retour de l’invite, puis redémarrez le service Livy à partir d’Ambari, ce qui devrait réussir.

> [!NOTE]
> `DELETE` la session Livy une fois son exécution terminée. Les sessions de traitement par lots Livy ne sont pas supprimées automatiquement dès que l’application Spark est terminée, ce qui est le cas par défaut. Une session Livy est une entité créée par une requête POST sur le serveur REST livy. Un appel `DELETE` est nécessaire pour supprimer cette entité. Ou nous devrions attendre que le GC démarre.

---

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* [Vue d’ensemble de la gestion de la mémoire dans Spark](https://spark.apache.org/docs/latest/tuning.html#memory-management-overview).

* [Débogage d’une application Spark sur des clusters HDInsight](/archive/blogs/azuredatalake/spark-debugging-101).

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).