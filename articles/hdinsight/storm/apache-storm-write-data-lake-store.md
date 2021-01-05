---
title: 'Tutoriel : HDInsight Apache Storm vers Stockage – Azure/Data Lake'
description: Didacticiel - Découvrez comment utiliser Apache Storm pour écrire dans le stockage compatible HDFS pour Azure HDInsight.
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 06/24/2019
ms.openlocfilehash: 8cdcc60e6eeec22e202d670c3fc057711bd432a6
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545444"
---
# <a name="tutorial-write-to-apache-hadoop-hdfs-from-apache-storm-on-azure-hdinsight"></a>Tutoriel : Écrire dans Apache Hadoop HDFS à partir d'Apache Storm sur Azure HDInsight

Ce didacticiel montre comment utiliser Apache Storm pour écrire des données dans le stockage compatible HDFS utilisé par Apache Storm sur HDInsight. HDInsight peut utiliser à la fois Stockage Azure et Azure Data Storage comme stockage compatible HDFS. Storm fournit un composant [HdfsBolt](https://storm.apache.org/releases/current/javadocs/org/apache/storm/hdfs/bolt/HdfsBolt.html) qui écrit des données dans le stockage HDFS. Ce document fournit des informations sur l’écriture dans ces deux types de stockages à partir du composant HdfsBolt.

L’exemple de topologie utilisé dans ce document s’appuie sur les composants inclus avec Storm sur HDInsight. Il faudra peut-être le modifier pour qu’il fonctionne avec Azure Data Lake Storage en cas d’utilisation avec d’autres clusters Apache Storm.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer le cluster avec l’action de script
> * Génération et empaquetage de la topologie
> * Déployer et exécuter la topologie
> * Affichage des données de sortie
> * Arrêt de la topologie

## <a name="prerequisites"></a>Prérequis

* [Kit de développeur Java (JDK) version 8](/azure/developer/java/fundamentals/java-jdk-long-term-support)

* [Apache Maven](https://maven.apache.org/download.cgi) correctement [installé](https://maven.apache.org/install.html) en fonction d’Apache.  Maven est un système de génération de projet pour les projets Java.

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

* Le [schéma d’URI](../hdinsight-hadoop-linux-information.md#URI-and-scheme) de votre principal espace de stockage de clusters. Il s’agirait de `wasb://` pour Stockage Azure, de `abfs://` pour Azure Data Lake Storage Gen2 ou de `adl://` pour Azure Data Lake Storage Gen1. Si le transfert sécurisé est activé pour le stockage Azure, l’URI sera `wasbs://`.  Voir aussi [transfert sécurisé](../../storage/common/storage-require-secure-transfer.md).

### <a name="example-configuration"></a>Exemple de configuration

Le code YAML suivant est un extrait du fichier `resources/writetohdfs.yaml` inclus dans cet exemple. Ce fichier définit la topologie de Storm à l’aide du framework [Flux](https://storm.apache.org/releases/current/flux.html) pour Apache Storm.

```yaml
components:
  - id: "syncPolicy"
    className: "org.apache.storm.hdfs.bolt.sync.CountSyncPolicy"
    constructorArgs:
      - 1000

  # Rotate files when they hit 5 MB
  - id: "rotationPolicy"
    className: "org.apache.storm.hdfs.bolt.rotation.FileSizeRotationPolicy"
    constructorArgs:
      - 5
      - "MB"

  - id: "fileNameFormat"
    className: "org.apache.storm.hdfs.bolt.format.DefaultFileNameFormat"
    configMethods:
      - name: "withPath"
        args: ["${hdfs.write.dir}"]
      - name: "withExtension"
        args: [".txt"]

  - id: "recordFormat"
    className: "org.apache.storm.hdfs.bolt.format.DelimitedRecordFormat"
    configMethods:
      - name: "withFieldDelimiter"
        args: ["|"]

# spout definitions
spouts:
  - id: "tick-spout"
    className: "com.microsoft.example.TickSpout"
    parallelism: 1


# bolt definitions
bolts:
  - id: "hdfs-bolt"
    className: "org.apache.storm.hdfs.bolt.HdfsBolt"
    configMethods:
      - name: "withConfigKey"
        args: ["hdfs.config"]
      - name: "withFsUrl"
        args: ["${hdfs.url}"]
      - name: "withFileNameFormat"
        args: [ref: "fileNameFormat"]
      - name: "withRecordFormat"
        args: [ref: "recordFormat"]
      - name: "withRotationPolicy"
        args: [ref: "rotationPolicy"]
      - name: "withSyncPolicy"
        args: [ref: "syncPolicy"]
```

Ce code YAML définit les éléments suivants :

* `syncPolicy`: définit quand les fichiers sont synchronisés/vidés dans le système de fichiers. Dans cet exemple, tous les 1 000 tuples.
* `fileNameFormat`: définit le modèle de chemin et de nom de fichier à utiliser lors de l’écriture de fichiers. Dans cet exemple, le chemin d’accès est fourni au moment de l’exécution à l’aide d’un filtre, avec l’extension de fichier `.txt`.
* `recordFormat`: définit le format interne des fichiers écrits. Dans cet exemple, les champs sont délimités par le caractère `|`.
* `rotationPolicy`: définit le moment de rotation des fichiers. Dans cet exemple, aucune rotation n’est effectuée.
* `hdfs-bolt`: utilise les composants précédents comme paramètres de configuration pour la classe `HdfsBolt`.

Pour plus d’informations sur le framework Flux, voir [https://storm.apache.org/releases/current/flux.html](https://storm.apache.org/releases/current/flux.html).

## <a name="configure-the-cluster"></a>Configurer le cluster

Par défaut, Storm sur HDInsight n’inclut pas les composants utilisés par `HdfsBolt` pour communiquer avec Stockage Azure ou Data Lake Storage dans le chemin de classe du cluster Storm. Utilisez l’action de script suivante pour ajouter ces composants au répertoire `extlib` de Storm sur votre cluster :

| Propriété | Valeur |
|---|---|
|Type de script |- Personnalisé|
|URI de script bash |`https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh`|
|Type(s) de nœud |Nimbus, Superviseur|
|Paramètres |None|

Pour plus d’informations sur l’utilisation de ce script avec votre cluster, consultez le document [Personnalisation de clusters HDInsight basés sur Linux à l’aide d'une action de script](./../hdinsight-hadoop-customize-cluster-linux.md).

## <a name="build-and-package-the-topology"></a>Génération et empaquetage de la topologie

1. Téléchargez l’exemple de projet à partir de [https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store](https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store) dans votre environnement de développement.

2. À partir d’une invite de commandes, d’un terminal ou d’une session shell, modifiez les répertoires à la racine du projet téléchargé. Pour générer et empaqueter la topologie, utilisez la commande suivante :

    ```cmd
    mvn compile package
    ```

    Une fois la génération et l’empaquetage terminés, vous obtenez un répertoire nommé `target`, qui contient un fichier `StormToHdfs-1.0-SNAPSHOT.jar`. Ce fichier contient la topologie compilée.

## <a name="deploy-and-run-the-topology"></a>Déployer et exécuter la topologie

1. Utilisez la commande suivante pour copier la topologie sur le cluster HDInsight. Remplacez `CLUSTERNAME` par le nom du cluster.

    ```cmd
    scp target\StormToHdfs-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:StormToHdfs-1.0-SNAPSHOT.jar
    ```

1. Une fois le téléchargement terminé, utilisez les éléments suivants pour vous connecter au cluster HDInsight à l’aide de SSH. Remplacez `CLUSTERNAME` par le nom du cluster.

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Une fois connecté, utilisez la commande suivante pour créer un fichier nommé `dev.properties` :

    ```bash
    nano dev.properties
    ```

1. Utilisez les données suivantes comme contenu du fichier `dev.properties`. Révisez si nécessaire en fonction de votre [schéma d’URI](../hdinsight-hadoop-linux-information.md#URI-and-scheme).

    ```
    hdfs.write.dir: /stormdata/
    hdfs.url: wasbs:///
    ```

    Pour enregistrer le fichier, use __Ctrl + X__ , puis __Y__ , et enfin __Entrée__ . Les valeurs dans ce fichier définissent l’URL de stockage et le nom du répertoire où sont écrites les données.

1. Utilisez la commande suivante pour démarrer la topologie :

    ```bash
    storm jar StormToHdfs-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /writetohdfs.yaml --filter dev.properties
    ```

    Cette commande démarre la topologie à partir du framework Flux en le soumettant au nœud Nimbus du cluster. La topologie est définie par le fichier `writetohdfs.yaml` inclus dans le fichier jar. Le fichier `dev.properties` est transféré en tant que filtre et les valeurs qu’il contient sont lues par la topologie.

## <a name="view-output-data"></a>Affichage des données de sortie

Pour afficher les données, utilisez la commande suivante :

  ```bash
  hdfs dfs -ls /stormdata/
  ```

Une liste des fichiers créés par cette topologie s’affiche. La liste suivante est un exemple des données retournées par les commandes précédentes :

```output
Found 23 items
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-0-1561407909895.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-1-1561407915577.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-10-1561407943327.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-11-1561407946312.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-12-1561407949320.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-13-1561407952662.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-14-1561407955502.txt
```

## <a name="stop-the-topology"></a>Arrêt de la topologie

Les topologies Storm s’exécutent jusqu’à ce qu’elles soient arrêtées ou que le cluster soit supprimé. Pour arrêter la topologie, utilisez la commande suivante :

```bash
storm kill hdfswriter
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les ressources créées par ce didacticiel, vous pouvez supprimer le groupe de ressources. La suppression du groupe de ressources efface également le cluster HDInsight associé et d’autres ressources liées au groupe de ressources.

Pour supprimer le groupe de ressources à l’aide du portail Azure :

1. Sur le portail Azure, développez le menu de gauche pour ouvrir le menu des services, et sélectionnez __Groupes de ressources__ pour afficher la liste de vos groupes de ressources.
2. Recherchez le groupe de ressources à supprimer, puis faites un clic droit sur le bouton __Plus__ (...) se trouvant à droite de la liste.
3. Sélectionnez __Supprimer le groupe de ressources__ et confirmez.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez découvert comment utiliser Apache Storm pour écrire des données dans le stockage compatible HDFS utilisé par Apache Storm sur HDInsight.

> [!div class="nextstepaction"]
> Découvrir d’autres [exemples Apache Storm pour HDInsight](apache-storm-example-topology.md)