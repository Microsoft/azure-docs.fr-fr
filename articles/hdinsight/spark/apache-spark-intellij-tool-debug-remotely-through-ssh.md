---
title: 'Azure Toolkit for IntelliJ : Déboguer des applications Spark avec SSH – HDInsight'
description: Obtenez des instructions étape par étape sur la façon d’utiliser HDInsight Tools dans le kit de ressources Azure pour IntelliJ pour déboguer des applications à distance sur des clusters HDInsight via SSH
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 12/23/2019
ms.openlocfilehash: 421993ac4aaba551b6fcbd002783d44559ce377d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995333"
---
# <a name="debug-apache-spark-applications-on-an-hdinsight-cluster-with-azure-toolkit-for-intellij-through-ssh"></a>Déboguez des applications Apache Spark sur un cluster HDInsight avec le kit de ressources Azure pour IntelliJ via SSH

Cet article propose des instructions étape par étape sur la façon d’utiliser HDInsight Tools dans le [kit de ressources Azure pour IntelliJ](/azure/developer/java/toolkit-for-intellij) afin de déboguer des applications à distance sur un cluster HDInsight. Pour déboguer votre projet, vous pouvez également regarder la vidéo [Debug HDInsight Spark applications with Azure Toolkit for IntelliJ](https://channel9.msdn.com/Series/AzureDataLake/Debug-HDInsight-Spark-Applications-with-Azure-Toolkit-for-IntelliJ) (Déboguer des applications HDInsight Spark avec le kit de ressources Azure pour IntelliJ).

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Spark sur HDInsight. Consultez [Créer un cluster Apache Spark](../spark/apache-spark-jupyter-spark-sql-use-portal.md).

* Pour les utilisateurs Windows : Quand vous exécutez l’application Spark Scala locale sur un ordinateur Windows, vous pouvez obtenir une exception, comme l’explique le document [SPARK-2356](https://issues.apache.org/jira/browse/SPARK-2356). Cette exception est liée à l’absence du fichier WinUtils.exe sur Windows.

    Pour résoudre cette erreur, téléchargez le fichier [Winutils.exe](https://github.com/steveloughran/winutils) à un emplacement tel que **C:\WinUtils\bin**. Ajoutez ensuite la variable d’environnement **HADOOP_HOME** et définissez la valeur de la variable sur **C:\WinUtils**.

* [IntelliJ IDEA](https://www.jetbrains.com/idea/download/#section=windows) (l’édition Community est gratuite.).

* [Kit de ressources Azure pour IntelliJ](/azure/developer/java/toolkit-for-intellij/installation).

* [Plug-in Scala pour IntelliJ](../spark/apache-spark-intellij-tool-plugin.md#install-scala-plugin-for-intellij-idea).

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="create-a-spark-scala-application"></a>Créer une application Spark Scala

1. Démarrez IntelliJ IDEA, puis sélectionnez **Create New Project** (Créer un projet) pour ouvrir la fenêtre **New Project** (Nouveau projet).

1. Sélectionnez **Apache Spark/HDInsight** dans le volet gauche.

1. Sélectionnez **Spark Project avec exemples (Scala)** [Projet Spark (Scala)] dans la fenêtre principale.

1. Dans la liste déroulante **Build tool** (Outil de build), sélectionnez l’un des outils suivants :

    * **Maven** pour la prise en charge de l’Assistant de création de projets Scala.
    * **SBT** pour gérer les dépendances et la génération du projet Scala.

     ![IntelliJ - Créer un projet Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-create-projectfor-debug-remotely.png)

1. Sélectionnez **Suivant**.

1. Dans la fenêtre suivante **New Project** (Nouveau projet), entrez les informations suivantes :

    |Propriété |Description |
    |---|---|
    |Nom du projet|Entrez un nom. Cette procédure utilise `myApp`.|
    |Emplacement du projet|Entrez l’emplacement où vous souhaitez enregistrer votre projet.|
    |Project SDK (SDK du projet)|Si vide, sélectionnez **New...** (Nouveau) et accédez à votre JDK.|
    |Version de Spark|L’Assistant de création intègre la version correcte des SDK Spark et Scala. Si la version du cluster Spark est antérieure à la version 2.0, sélectionnez **Spark 1.x**. Sinon, sélectionnez **Spark 2.x.** . Cet exemple utilise **Spark 2.3.0 (Scala 2.11.8)** .|

   ![Nouveau projet IntelliJ - Sélection de la version Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-new-project.png)

1. Sélectionnez **Terminer**. Vous devrez peut-être patienter quelques minutes avant que le projet soit disponible. La progression s’affiche dans l’angle inférieur droit.

1. Développez votre projet et accédez à **src** > **main** > **scala** > **sample**. Double-cliquez sur **SparkCore_WasbIOTest**.

## <a name="perform-local-run"></a>Effectuer une exécution locale

1. Dans le script **SparkCore_wasbloTest**, cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez l’option **SparkCore_WasbIOTest** pour effectuer l’exécution locale.

1. Une fois l’exécution locale terminée, vous pouvez voir le fichier de sortie enregistré dans l’Explorateur de votre projet actuel **data** >  **__default__** .

    ![Intellij - Résultat d'exécution locale du projet](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/spark-local-run-result.png)

1. Nos outils définissent la configuration de l’exécution locale par défaut automatiquement lorsque vous effectuez l’exécution et le débogage locaux. Ouvrez la configuration **[Spark on HDInsight] XXX** en haut à droite. Comme vous pouvez le voir, la configuration **[Spark on HDInsight]XXX** est déjà créée sous **Apache Spark on HDInsight**. Basculez vers l’onglet **Locally Run (Exécuter localement)** .

    ![Intellij - Exécution locale des configuration de débogage](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/local-run-configuration.png)

    - [Variables d’environnement](#prerequisites) : si vous avez déjà défini la variable d’environnement système **HADOOP_HOME** sur **C:\WinUtils**, elle peut détecter automatiquement qu’aucun ajout manuel n’est nécessaire.
    - [Emplacement de WinUtils.exe](#prerequisites) : si vous n’avez pas défini la variable d’environnement système, vous pouvez trouver l’emplacement en cliquant sur son bouton.
    - Choisissez simplement l’une des deux options ; elles ne sont pas requises sur MacOS et Linux.

1. Vous pouvez également définir la configuration manuellement avant d’effectuer l’exécution et le débogage locaux. Dans la capture d’écran précédente, sélectionnez le signe plus ( **+** ). Sélectionnez ensuite l’option **Apache Spark on HDInsight**. Entrez les informations de **Nom** et **Nom de la classe principale** à enregistrer, puis cliquez sur le bouton d’exécution locale.

## <a name="perform-local-debugging"></a>Effectuer un débogage local

1. Ouvrez le script **SparkCore_wasbloTest**, et définissez les points d’arrêt.

1. Cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez l’option **Debug '[Spark on HDInsight]XXX'** pour effectuer le débogage local.

## <a name="perform-remote-run"></a>Effectuer une exécution à distance

1. Accédez à **Exécuter** > **Modifier les configurations...** . Dans ce menu, vous pouvez créer ou modifier les configurations pour le débogage à distance.

1. Dans la boîte de dialogue **Run/Debug Configurations** (Exécuter/Déboguer les configurations) sélectionnez le signe plus ( **+** ). Sélectionnez ensuite l’option **Apache Spark on HDInsight**.

   ![IntelliJ - Ajouter une configuration](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-add-new-Configuration.png)

1. Basculez vers l’onglet **Remotely Run in Cluster (Exécuter à distance dans le cluster)** . Entrez les informations sur le **Nom**, le **Cluster Spark** et le **Nom principal de la classe**. Cliquez ensuite sur **Advanced configuration (Remote Debugging)** [Configuration avancée (Débogage à distance)]. Nos outils prennent en charge le débogage avec **Exécuteurs**. Pour **numExectors**, la valeur par défaut est 5. Il est déconseillé de définir une valeur supérieure à 3.

   ![Intellij - Exécuter le débogage des configurations](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-run-debug-configurations.png)

1. Dans la section **Advanced Configuration (Remote Debugging)** , sélectionnez **Enable Spark remote debug**. Entrez le nom d’utilisateur SSH, puis entrez un mot de passe ou utilisez un fichier de clé privée. Si vous souhaitez effectuer le débogage à distance, vous devez définir cette option. Par contre, cette action n’est pas nécessaire si vous souhaitez simplement utiliser l’exécution à distance.

   ![Configuration avancée Intellij - Activer le débogage à distance Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-enable-spark-remote-debug.png)

1. La configuration est maintenant enregistrée avec le nom fourni. Pour afficher les détails de configuration, sélectionnez le nom de configuration. Pour apporter des modifications, sélectionnez **Modifier les configurations**.

1. Une fois le paramétrage des configurations terminé, vous pouvez exécuter le projet sur le cluster à distance ou effectuer un débogage à distance.

   ![IntelliJ - Bouton d’exécution du débogage à distance des travaux Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/perform-remote-run-button.png)

1. Cliquez sur le bouton **Déconnecter** si les journaux d’activité d’envoi n’apparaissent pas dans le panneau gauche. Toutefois, une exécution a toujours lieu sur le serveur principal.

   ![IntelliJ - Résultat de l’exécution du débogage à distance des travaux Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/spark-remote-run-result.png)

## <a name="perform-remote-debugging"></a>Effectuer un débogage à distance

1. Configurez des points de rupture, puis sélectionnez l’icône **Débogage distant**. La différence avec la soumission à distance est que le nom d’utilisateur/mot de passe SSH doit être configuré.

   ![IntelliJ - Icône de débogage à distance des travaux Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debug-icon.png)

1. Quand l’exécution du programme atteint le point de rupture, un onglet **Pilote** et deux onglets **Exécuteur** s’affichent dans le volet **Débogueur**. Sélectionnez l’icône **Reprendre le programme** pour continuer à exécuter le code ; le point de rupture suivant est alors atteint. Vous devez basculer vers l’onglet **Exécuteur** correct pour trouver l’exécuteur cible à déboguer. Vous pouvez consulter les journaux d’activité d’exécution sous l’onglet **Console** correspondant.

   ![IntelliJ - Onglet de débogage à distance des travaux Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debugger-tab.png)

### <a name="perform-remote-debugging-and-bug-fixing"></a>Effectuer un débogage et une résolution des bogues à distance

1. Configurez deux points de rupture, puis sélectionnez l’icône **Déboguer** pour démarrer le processus de débogage à distance.

1. Le code s’arrête au premier point de rupture et les informations sur les paramètres et les variables s’affichent dans le volet **Variable**.

1. Sélectionnez l’icône **Reprendre le programme** pour continuer. Le code s’arrête au deuxième point. L’exception est interceptée comme prévu.

   ![IntelliJ - Lever l'erreur du débogage à distance des travaux Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-throw-error.png)

1. Resélectionnez l’icône **Reprendre le programme**. La fenêtre **Soumission HDInsight Spark** affiche une erreur d’exécution de tâche.

   ![IntelliJ - Soumission de l'erreur du débogage à distance des travaux Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-error-submission.png)

1. Pour mettre à jour dynamiquement la valeur de la variable à l’aide de la fonctionnalité de débogage IntelliJ, resélectionnez **Déboguer**. Le volet **Variables** réapparaît.

1. Cliquez avec le bouton droit de la cible de l’onglet **Déboguer**, puis sélectionnez **Définir la valeur**. Ensuite, entrez une nouvelle valeur pour la variable. Puis sélectionnez **Entrer** pour enregistrer la valeur.

   ![IntelliJ - Valeur de définition de débogage à distance des travaux Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-set-value1.png)

1. Sélectionnez l’icône **Reprendre le programme** pour continuer à exécuter le programme. Cette fois-ci, aucune exception n’est interceptée. Vous pouvez voir que le projet s’exécute correctement sans aucune exception.

   ![IntelliJ - Débogage à distance des travaux Spark sans exception](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debug-without-exception.png)

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble : Apache Spark sur Azure HDInsight](apache-spark-overview.md)

### <a name="demo"></a>Démonstration

* Créer un projet Scala (vidéo) : [Créer des applications Apache Spark Scala](https://channel9.msdn.com/Series/AzureDataLake/Create-Spark-Applications-with-the-Azure-Toolkit-for-IntelliJ)
* Débogage à distance (vidéo) : [Utiliser Azure Toolkit for IntelliJ pour déboguer des applications Apache Spark à distance sur un cluster HDInsight](https://channel9.msdn.com/Series/AzureDataLake/Debug-HDInsight-Spark-Applications-with-Azure-Toolkit-for-IntelliJ)

### <a name="scenarios"></a>Scénarios

* [Apache Spark avec BI : Effectuer une analyse interactive des données à l’aide de Spark dans HDInsight avec les outils décisionnels](apache-spark-use-bi-tools.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour analyser la température de bâtiments à l’aide des données des systèmes HVAC](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour prédire les résultats de l’inspection d’aliments](apache-spark-machine-learning-mllib-ipython.md)
* [Analyse des journaux de site web à l’aide d’Apache Spark dans HDInsight](./apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>Création et exécution d’applications

* [Créer une application autonome avec Scala](./apache-spark-create-standalone-application.md)
* [Exécuter des tâches à distance avec Apache Livy sur un cluster Apache Spark](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>Outils et extensions

* [Utiliser Azure Toolkit for IntelliJ afin de créer des applications Apache Spark pour un cluster HDInsight](apache-spark-intellij-tool-plugin.md)
* [Utiliser Azure Toolkit for IntelliJ pour déboguer des applications Apache Spark à distance par VPN](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Utiliser HDInsight Tools dans Azure Toolkit for Eclipse pour créer des applications Apache Spark](./apache-spark-eclipse-tool-plugin.md)
* [Utiliser des blocs-notes Apache Zeppelin avec un cluster Apache Spark sur HDInsight](apache-spark-zeppelin-notebook.md)
* [Noyaux disponibles pour le bloc-notes Jupyter dans le cluster Apache Spark pour HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Utiliser des packages externes avec les blocs-notes Jupyter](apache-spark-jupyter-notebook-use-external-packages.md)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster (Installer Jupyter sur un ordinateur et se connecter au cluster Spark sur HDInsight)](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>Gestion des ressources

* [Gérer les ressources du cluster Apache Spark dans Azure HDInsight](apache-spark-resource-manager.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight (Suivi et débogage des tâches en cours d’exécution sur un cluster Apache Spark dans HDInsight)](apache-spark-job-debugging.md)