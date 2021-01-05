---
title: Workflows Apache Oozie et Sécurité Entreprise - Azure HDInsight
description: Sécurisez les workflows Apache Oozie à l’aide du Pack Sécurité Entreprise Azure HDInsight. Découvrez comment définir un workflow Oozie et envoyer un travail Oozie.
author: omidm1
ms.author: omidm
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,seodec18,seoapr2020
ms.date: 05/14/2020
ms.openlocfilehash: 71432dc2dd09381d39ca646d45c7f2bdd96012c6
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97653433"
---
# <a name="run-apache-oozie-in-azure-hdinsight-clusters-with-enterprise-security-package"></a>Exécuter Apache Oozie dans des clusters Azure HDInsight avec le Pack Sécurité Entreprise

Apache Oozie est un workflow et un système de coordination qui gère les tâches Apache Hadoop. Oozie est intégré dans la pile Hadoop et prend en charge les travaux suivants :

- Apache MapReduce
- Apache Pig
- Apache Hive
- Apache Sqoop

Vous pouvez également utiliser Oozie pour planifier des travaux propres à un système, comme des programmes Java ou des scripts de l’interpréteur de commandes.

## <a name="prerequisite"></a>Configuration requise

Un cluster Azure HDInsight Hadoop avec le Pack Sécurité Entreprise (ESP). Voir [Configurer des clusters HDInsight avec ESP](./apache-domain-joined-configure-using-azure-adds.md).

> [!NOTE]  
> Pour obtenir des instructions détaillées sur la façon d’utiliser Oozie dans des clusters non-ESP, consultez [Utiliser des workflows Apache Oozie dans Azure HDInsight Linux](../hdinsight-use-oozie-linux-mac.md).

## <a name="connect-to-an-esp-cluster"></a>Se connecter à un cluster ESP

Pour en savoir plus sur Secure Shell (SSH), consultez la page [Se connecter à HDInsight (Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

1. Connectez-vous au cluster HDInsight à l’aide de SSH :

    ```bash
    ssh [DomainUserName]@<clustername>-ssh.azurehdinsight.net
    ```

1. Pour vérifier si l’authentification Kerberos a réussi, utilisez la commande `klist`. Si ce n’est pas le cas, utilisez `kinit` pour lancer l’authentification Kerberos.

1. Connectez-vous à la passerelle HDInsight pour inscrire le jeton OAuth requis pour l’accès à Azure Data Lake Storage (ADLS) :

    ```bash
    curl -I -u [DomainUserName@Domain.com]:[DomainUserPassword] https://<clustername>.azurehdinsight.net
    ```

    Le code de réponse d’état **200 OK** indique la réussite de l’inscription. Si vous recevez une erreur 401 (inscription non autorisée), vérifiez le nom d’utilisateur et le mot de passe.

## <a name="define-the-workflow"></a>Définition du flux de travail

Les définitions des workflows Oozie sont écrites en hPDL (langage de définition de processus Apache Hadoop). hPDL est un langage de définition de processus XML. Pour définir le workflow, procédez comme suit :

1. Configurez l’espace de travail de l’utilisateur du domaine :

   ```bash
   hdfs dfs -mkdir /user/<DomainUser>
   cd /home/<DomainUserPath>
   cp /usr/hdp/<ClusterVersion>/oozie/doc/oozie-examples.tar.gz .
   tar -xvf oozie-examples.tar.gz
   hdfs dfs -put examples /user/<DomainUser>/
   ```

   Remplacez `DomainUser` par le nom de l’utilisateur de domaine.
   Remplacez `DomainUserPath` par le chemin du répertoire de base de l’utilisateur de domaine.
   Remplacez `ClusterVersion` par votre version de plateforme de données de cluster.

2. Pour créer et modifier un fichier, utilisez l’instruction suivante :

   ```bash
   nano workflow.xml
   ```

3. Une fois que l’éditeur nano est ouvert, saisissez le code XML suivant comme contenu du fichier :

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <workflow-app xmlns="uri:oozie:workflow:0.4" name="map-reduce-wf">
       <credentials>
          <credential name="metastore_token" type="hcat">
             <property>
                <name>hcat.metastore.uri</name>
                <value>thrift://<active-headnode-name>-<clustername>.<Domain>.com:9083</value>
             </property>
             <property>
                <name>hcat.metastore.principal</name>
                <value>hive/_HOST@<Domain>.COM</value>
             </property>
          </credential>
          <credential name="hs2-creds" type="hive2">
             <property>
                <name>hive2.server.principal</name>
                <value>${jdbcPrincipal}</value>
             </property>
             <property>
                <name>hive2.jdbc.url</name>
                <value>${jdbcURL}</value>
             </property>
          </credential>
       </credentials>
       <start to="mr-test" />
       <action name="mr-test">
          <map-reduce>
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <prepare>
                <delete path="${nameNode}/user/${wf:user()}/examples/output-data/mrresult" />
             </prepare>
             <configuration>
                <property>
                   <name>mapred.job.queue.name</name>
                   <value>${queueName}</value>
                </property>
                <property>
                   <name>mapred.mapper.class</name>
                   <value>org.apache.oozie.example.SampleMapper</value>
                </property>
                <property>
                   <name>mapred.reducer.class</name>
                   <value>org.apache.oozie.example.SampleReducer</value>
                </property>
                <property>
                   <name>mapred.map.tasks</name>
                   <value>1</value>
                </property>
                <property>
                   <name>mapred.input.dir</name>
                   <value>/user/${wf:user()}/${examplesRoot}/input-data/text</value>
                </property>
                <property>
                   <name>mapred.output.dir</name>
                   <value>/user/${wf:user()}/${examplesRoot}/output-data/mrresult</value>
                </property>
             </configuration>
          </map-reduce>
          <ok to="myHive2" />
          <error to="fail" />
       </action>
       <action name="myHive2" cred="hs2-creds">
          <hive2 xmlns="uri:oozie:hive2-action:0.2">
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <jdbc-url>${jdbcURL}</jdbc-url>
             <script>${hiveScript2}</script>
             <param>hiveOutputDirectory2=${hiveOutputDirectory2}</param>
          </hive2>
          <ok to="myHive" />
          <error to="fail" />
       </action>
       <action name="myHive" cred="metastore_token">
          <hive xmlns="uri:oozie:hive-action:0.2">
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <configuration>
                <property>
                   <name>mapred.job.queue.name</name>
                   <value>${queueName}</value>
                </property>
             </configuration>
             <script>${hiveScript1}</script>
             <param>hiveOutputDirectory1=${hiveOutputDirectory1}</param>
          </hive>
          <ok to="end" />
          <error to="fail" />
       </action>
       <kill name="fail">
          <message>Oozie job failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
       </kill>
       <end name="end" />
    </workflow-app>
    ```

4. Remplacez `clustername` par le nom du cluster.

5. Pour enregistrer le fichier, appuyez sur **Ctrl+X**. Entrez **Y**. Ensuite, appuyez sur **Entrée**.

    Le workflow comprend deux parties :

   - **Informations d'identification.** Cette section comprend les informations d’identification utilisées pour l’authentification des actions Oozie :

     Cet exemple utilise l’authentification des actions Hive. Pour en savoir plus, voir l’[authentification des actions](https://oozie.apache.org/docs/4.2.0/DG_ActionAuthentication.html).

     Le service d’informations d’identification permet aux actions Oozie d’emprunter l’identité de l’utilisateur pour accéder aux services Hadoop.

   - **Action.** Cette section comporte trois actions : map-reduce, Hive server 2 et Hive server 1 :

     - L’action map-reduce exécute un exemple de package Oozie pour map-reduce qui génère le nombre de mots agrégés.

     - Les actions Hive server 2 et Hive server 1 exécutent une requête sur l’échantillon de table Hive fourni avec HDInsight.

     Les actions Hive utilisent les informations d’identification définies dans la section Informations d’identification en vue d’une authentification à l’aide du mot clé `cred` dans l’élément d’action.

6. Utilisez la commande suivante pour copier le fichier `workflow.xml` vers `/user/<domainuser>/examples/apps/map-reduce/workflow.xml` :

    ```bash
    hdfs dfs -put workflow.xml /user/<domainuser>/examples/apps/map-reduce/workflow.xml
    ```

7. Remplacez `domainuser` par le nom d’utilisateur de votre domaine.

## <a name="define-the-properties-file-for-the-oozie-job"></a>Définir le fichier de propriétés de la tâche Oozie

1. Pour créer et modifier un fichier de propriétés de tâche, utilisez l’instruction suivante :

    ```bash
    nano job.properties
    ```

2. Une fois que l’éditeur nano est ouvert, utilisez le code XML suivant comme contenu du fichier :

   ```bash
   nameNode=adl://home
   jobTracker=headnodehost:8050
   queueName=default
   examplesRoot=examples
   oozie.wf.application.path=${nameNode}/user/[domainuser]/examples/apps/map-reduce/workflow.xml
   hiveScript1=${nameNode}/user/${user.name}/countrowshive1.hql
   hiveScript2=${nameNode}/user/${user.name}/countrowshive2.hql
   oozie.use.system.libpath=true
   user.name=[domainuser]
   jdbcPrincipal=hive/<active-headnode-name>.<Domain>.com@<Domain>.COM
   jdbcURL=[jdbcurlvalue]
   hiveOutputDirectory1=${nameNode}/user/${user.name}/hiveresult1
   hiveOutputDirectory2=${nameNode}/user/${user.name}/hiveresult2
   ```

   - Utilisez l'URI `adl://home` de la propriété `nameNode` si vous disposez d'Azure Data Lake Storage Gen1 comme espace de stockage en cluster principal. Si vous utilisez le service Stockage Blob Azure, remplacez cette valeur par `wasb://home`. Si vous utilisez Azure Data Lake Storage Gen2, remplacez cette valeur par `abfs://home`.
   - Remplacez `domainuser` par le nom d’utilisateur de votre domaine.
   - Remplacez `ClusterShortName` par le nom court du cluster. Par exemple, si le nom du cluster est https:// *[exemple de lien]* sechadoopcontoso.azurehdisnight.net, le `clustershortname` est les six premiers caractères du cluster : **sechad**.
   - Remplacez `jdbcurlvalue` par l’URL JDBC de la configuration Hive. Par exemple, jdbc:hive2://headnodehost:10001/;transportMode=http.
   - Pour enregistrer le fichier, sélectionnez Ctrl + X, entrez `Y`, puis sélectionnez **Entrée**.

   Ce fichier de propriétés doit être présent localement lors de l’exécution des tâches Oozie.

## <a name="create-custom-hive-scripts-for-oozie-jobs"></a>Créer des scripts Hive personnalisés pour les tâches Oozie

Vous pouvez créer les deux scripts Hive pour les actions Hive server 2 et Hive server 1 comme indiqué dans les sections suivantes.

### <a name="hive-server-1-file"></a>Fichier Hive server 1

1. Créez et modifiez un fichier pour les actions Hive server 1 :

    ```bash
    nano countrowshive1.hql
    ```

2. Créez le script :

    ```sql
    INSERT OVERWRITE DIRECTORY '${hiveOutputDirectory1}'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    select devicemake from hivesampletable limit 2;
    ```

3. Enregistrez le fichier dans le système de fichiers DFS Apache Hadoop (HDFS) :

    ```bash
    hdfs dfs -put countrowshive1.hql countrowshive1.hql
    ```

### <a name="hive-server-2-file"></a>Fichier Hive Server 2

1. Créez et modifiez un fichier pour les actions Hive server 2 :

    ```bash
    nano countrowshive2.hql
    ```

2. Créez le script :

    ```sql
    INSERT OVERWRITE DIRECTORY '${hiveOutputDirectory1}' 
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
    select devicemodel from hivesampletable limit 2;
    ```

3. Enregistrez le fichier dans HDFS :

    ```bash
    hdfs dfs -put countrowshive2.hql countrowshive2.hql
    ```

## <a name="submit-oozie-jobs"></a>Soumettre des tâches Oozie

La soumission de tâches Oozie pour les clusters ESP fonctionne de la même façon que la soumission de tâches Oozie pour les clusters non-ESP.

Pour plus d’informations, consultez [Utiliser Apache Oozie avec Apache Hadoop pour définir et exécuter un workflow Azure HDInsight basé sur Linux](../hdinsight-use-oozie-linux-mac.md).

## <a name="results-from-an-oozie-job-submission"></a>Résultats de la soumission d’une tâche Oozie

Les tâches Oozie sont exécutées pour l’utilisateur. Par conséquent, les journaux d’audit Apache Hadoop YARN et Apache Ranger affichent les travaux en cours d’exécution comme l’utilisateur impersonné. La sortie de l’interface de ligne de commande d’une tâche Oozie se présente comme le code suivant :

```output
Job ID : 0000015-180626011240801-oozie-oozi-W
------------------------------------------------------------------------------------------------
Workflow Name : map-reduce-wf
App Path      : adl://home/user/alicetest/examples/apps/map-reduce/wf.xml
Status        : SUCCEEDED
Run           : 0
User          : alicetest
Group         : -
Created       : 2018-06-26 19:25 GMT
Started       : 2018-06-26 19:25 GMT
Last Modified : 2018-06-26 19:30 GMT
Ended         : 2018-06-26 19:30 GMT
CoordAction ID: -

Actions
------------------------------------------------------------------------------------------------
ID                        Status    Ext ID            ExtStatus                 ErrCode
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@:start:    OK    -                         OK             -
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@mr-test    OK    job_1529975666160_0051    SUCCEEDED      -
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@myHive2    OK    job_1529975666160_0053    SUCCEEDED      -
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@myHive    OK     job_1529975666160_0055    SUCCEEDED      -
------------------------------------------------------------------------------------------------
0000015-180626011240801-oozie-oozi-W@end       OK     -                         OK             -
-----------------------------------------------------------------------------------------------
```

Les journaux d’audit Ranger pour les actions Hive server 2 montrent l’exécution d’une action par Oozie à la place de l’utilisateur. Seul l’administrateur du cluster peut voir les vues Ranger et YARN.

## <a name="configure-user-authorization-in-oozie"></a>Configurer l’autorisation utilisateur dans Oozie

Oozie permet de configurer l’autorisation utilisateur de manière à empêcher l’arrêt ou la suppression des tâches des autres utilisateurs. Pour activer cette configuration, définissez `oozie.service.AuthorizationService.security.enabled` sur `true`. 

Pour plus d’informations, consultez [Oozie Installation and Configuration](https://oozie.apache.org/docs/3.2.0-incubating/AG_Install.html).

Pour les composants tels que Hive server 1, où le plug-in Ranger n’est pas disponible ou pris en charge, seule l’autorisation HDFS de granularité grossière est possible. L’autorisation de granularité fine est uniquement disponible via les plug-ins Ranger.

## <a name="get-the-oozie-web-ui"></a>Obtenir l’interface utilisateur web Oozie

L’interface utilisateur web Oozie fournit une vue web de l’état des travaux Oozie sur le cluster. Pour obtenir l’interface utilisateur web, effectuez les étapes suivantes dans les clusters ESP :

1. Ajoutez un [nœud de périphérie](../hdinsight-apps-use-edge-node.md) et activez l’[authentification Kerberos SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

2. Suivre les étapes de [l’interface utilisateur web Oozie](../hdinsight-use-oozie-linux-mac.md) pour activer le tunneling SSH vers le nœud de périphérie et accéder à l’interface utilisateur web

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser Apache Oozie avec Apache Hadoop pour définir et exécuter un workflow sur Azure HDInsight Linux](../hdinsight-use-oozie-linux-mac.md).
- [Se connecter à HDInsight (Apache Hadoop) avec SSH](../hdinsight-hadoop-linux-use-ssh-unix.md#authentication-domain-joined-hdinsight).
