---
title: Résolution des problèmes liés au cluster Apache Spark dans Azure HDInsight
description: En savoir plus sur les problèmes liés aux clusters Apache Spark dans Azure HDInsight et comment les résoudre.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: troubleshooting
ms.date: 08/15/2019
ms.author: hrasheed
ms.openlocfilehash: 1dbf6478a62675c8b514298007a7663239d8f7cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86084640"
---
# <a name="known-issues-for-apache-spark-cluster-on-hdinsight"></a>Problèmes connus du cluster Apache Spark sur Azure HDInsight

Ce document fait le suivi de tous les problèmes connus pour la version Preview publique de HDInsight Spark.  

## <a name="apache-livy-leaks-interactive-session"></a>Une session interactive a fuité d’Apache Livy
Lorsqu’[Apache Livy](https://livy.incubator.apache.org/) est redémarré (à partir d’[Apache Ambari](https://ambari.apache.org/) ou à cause d’un redémarrage de la machine virtuelle du nœud principal 0) avec une session interactive encore active, une session de travail interactive fuite. Par conséquent, les nouveaux travaux peuvent être bloqués dans l’état Accepté.

**Atténuation :**

Pour contourner ce problème, suivez la procédure ci-dessous :

1. SSH dans le nœud principal. Pour plus d’informations, consultez [Utiliser SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

2. Exécutez la commande suivante pour rechercher l’ID d’application des tâches interactives démarrées via Livy.

   ```bash
   yarn application –list
   ```

    Par défaut, le nom attribué à une tâche est « Livy » si celle-ci a été démarrée avec une session interactive Livy, sans qu’un nom ait été explicitement spécifié. Pour la session Livy démarrée par [Jupyter Notebook](https://jupyter.org/), le nom de la tâche commence par `remotesparkmagics_*`.

3. Exécutez la commande suivante pour mettre fin à ces tâches.

   ```bash
   yarn application –kill <Application ID>
   ```

L’exécution des nouvelles tâches commence alors.

## <a name="spark-history-server-not-started"></a>Serveur d’historique Spark non démarré
Le serveur d’historique Spark ne démarre pas automatiquement après la création d’un cluster.  

**Atténuation :**

Démarrez manuellement le serveur d’historique à partir d’Ambari.

## <a name="permission-issue-in-spark-log-directory"></a>Problème d’autorisation dans le répertoire des journaux Spark
hdiuser obtient l’erreur suivante lors de l’envoi d’un travail à l’aide de spark-submit :

```
java.io.FileNotFoundException: /var/log/spark/sparkdriver_hdiuser.log (Permission denied)
```

Et aucun journal de pilotes n’est écrit.

**Atténuation :**

1. Ajoutez hdiuser au groupe Hadoop.
2. Fournissez les autorisations 777 sur /var/log/spark après la création du cluster.
3. Mettez à jour l’emplacement du journal Spark à l’aide d’Ambari pour obtenir un répertoire avec les autorisations 777.  
4. Exécutez spark-submit en tant que sudo.  

## <a name="spark-phoenix-connector-is-not-supported"></a>Le connecteur Spark-Phoenix n’est pas pris en charge.

Les clusters HDInsight Spark ne prennent pas en charge le connecteur Spark-Phoenix.

**Atténuation :**

Vous devez plutôt utiliser le connecteur Spark-HBase. Pour savoir comment procéder, consultez [How to use Spark-HBase connector](https://web.archive.org/web/20190112153146/https://blogs.msdn.microsoft.com/azuredatalake/2016/07/25/hdinsight-how-to-use-spark-hbase-connector/).

## <a name="issues-related-to-jupyter-notebooks"></a>Problèmes liés aux notebooks Jupyter

Voici certains problèmes connus liés aux notebooks Jupyter.

### <a name="notebooks-with-non-ascii-characters-in-filenames"></a>Notebooks avec des caractères non-ASCII dans les noms de fichiers

N’utilisez pas de caractères non-ASCII dans les noms de fichiers des bloc-notes Jupyter. Si vous essayez de charger un fichier via l’IU Jupyter, qui a un nom de fichier non-ASCII, le téléchargement échoue sans message d’erreur. Jupyter ne vous permet pas de charger le fichier, mais il ne lance pas d’erreur visible non plus.

### <a name="error-while-loading-notebooks-of-larger-sizes"></a>Erreur lors du chargement de notebooks de taille supérieure

Vous pouvez obtenir une erreur **`Error loading notebook`** lorsque vous tentez de charger un bloc-notes de taille supérieure.  

**Atténuation :**

Si vous obtenez cette erreur, cela ne signifie pas que vos données sont endommagées ou perdues.  Vos blocs-notes sont toujours sur le disque, sous `/var/lib/jupyter`et vous pouvez exécuter SSH dans le cluster pour y accéder. Pour plus d’informations, consultez [Utiliser SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

Une fois connecté au cluster à l’aide de SSH, vous pouvez copier les blocs-notes depuis le cluster vers votre ordinateur local (à l’aide de SCP ou WinSCP) pour en faire une sauvegarde afin d’éviter la perte de toutes les données importantes dans le bloc-notes. Vous pouvez ensuite créer un tunnel SSH dans votre nœud principal sur le port 8001, afin d’accéder à Jupyter sans avoir à passer par la passerelle.  À partir de là, vous pouvez effacer la sortie de votre bloc-notes et l’enregistrer de nouveau pour réduire au maximum la taille du bloc-notes.

Pour éviter cette erreur à l’avenir, gardez en tête les conseils suivants :

* Il est important que la taille du bloc-notes reste réduite. Aucune sortie de vos travaux sous Spark qui est envoyée à Jupyter n’est conservée dans le bloc-notes.  Pour Jupyter, la meilleure pratique consiste généralement à éviter l’exécution de l’élément `.collect()` sur des RDD ou trames de données larges. En revanche, si vous souhaitez lire le contenu d’un RDD, pensez à exécuter `.take()` ou `.sample()` afin que votre sortie ne devienne pas trop volumineuse.
* En outre, lorsque vous enregistrez un bloc-notes, supprimez toutes les cellules de sortie pour réduire sa taille.

### <a name="notebook-initial-startup-takes-longer-than-expected"></a>Démarrage du bloc-notes plus long que prévu

La première instruction de code du bloc-notes Jupyter avec Spark Magic peut nécessiter plusieurs minutes.  

**Explication :**

Cela se produit lorsque la première cellule du code est exécutée. En arrière-plan, la configuration de la session est lancée, et les contextes Spark, SQL et Hive sont définis. Une fois ces contextes définis, la première instruction est exécutée et donne l'impression que l'instruction prend beaucoup de temps.

### <a name="jupyter-notebook-timeout-in-creating-the-session"></a>Délai d’attente du bloc-notes Jupyter lors de la création de la session

Lorsque le cluster Spark manque de ressources, les noyaux Spark et PySpark du bloc-notes Jupyter expirent en essayant de créer la session.

**Atténuations :**

1. Libérez certaines ressources de votre cluster Spark :

   * Arrêtez les autres blocs-notes Spark en allant dans le menu Fermer et Arrêter ou en cliquant sur Arrêter dans l’explorateur du bloc-notes.
   * Arrêtez les autres applications Spark à partir de YARN.

2. Redémarrez le bloc-notes que vous tentiez de démarrer. Un nombre suffisant de ressources devrait désormais être disponible pour vous permettre de créer une session.

## <a name="see-also"></a>Voir aussi

* [Vue d’ensemble : Apache Spark sur Azure HDInsight](apache-spark-overview.md)

### <a name="scenarios"></a>Scénarios

* [Apache Spark avec BI : effectuer une analyse interactive des données à l’aide de Spark sur HDInsight avec des outils décisionnels](apache-spark-use-bi-tools.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour l’analyse de la température de bâtiments à l’aide des données des systèmes HVAC](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour prédire les résultats de l’inspection d’aliments](apache-spark-machine-learning-mllib-ipython.md)
* [Analyse des journaux de site web à l’aide d’Apache Spark dans HDInsight](apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>Création et exécution d’applications

* [Créer une application autonome avec Scala](apache-spark-create-standalone-application.md)
* [Exécuter des tâches à distance avec Apache Livy sur un cluster Apache Spark](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>Outils et extensions

* [Utilisation du plugin d’outils HDInsight pour IntelliJ IDEA pour créer et soumettre des applications Spark Scala](apache-spark-intellij-tool-plugin.md)
* [Utiliser le plug-in Azure HDInsight Tools pour IntelliJ IDEA afin de déboguer des applications Apache Spark à distance](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Utiliser des blocs-notes Apache Zeppelin avec un cluster Apache Spark sur HDInsight](apache-spark-zeppelin-notebook.md)
* [Noyaux accessibles à Jupyter Notebook dans le cluster Apache Spark pour HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Utiliser des packages externes avec les blocs-notes Jupyter](apache-spark-jupyter-notebook-use-external-packages.md)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster (Installer Jupyter sur un ordinateur et se connecter au cluster Spark sur HDInsight)](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>Gestion des ressources

* [Gérer les ressources du cluster Apache Spark dans Azure HDInsight](apache-spark-resource-manager.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight (Suivi et débogage des tâches en cours d’exécution sur un cluster Apache Spark dans HDInsight)](apache-spark-job-debugging.md)
