---
title: Déboguer des travaux Apache Spark en cours d’exécution sur Azure HDInsight
description: Utilisez l’interface utilisateur YARN, l’interface utilisateur Spark et le serveur d’historique Spark pour suivre et déboguer les tâches en cours d’exécution sur un cluster Spark dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: 817b8976f5d014d990945816c2df1c7e8ed729fb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86084883"
---
# <a name="debug-apache-spark-jobs-running-on-azure-hdinsight"></a>Déboguer des travaux Apache Spark en cours d’exécution sur Azure HDInsight

Dans cet article, vous allez apprendre à suivre et déboguer les travaux Apache Spark s’exécutant sur des clusters HDInsight. Déboguez à l’aide de l’interface utilisateur d’Apache Hadoop YARN, de l’interface utilisateur de Spark et du serveur d’historique Spark. Vous démarrez une tâche Spark à partir d’un bloc-notes disponible avec le cluster Spark, **Machine Learning : analyse prédictive des données d’inspections alimentaires à l’aide de MLLib**. Utilisez les étapes ci-dessous pour effectuer le suivi d’une application que vous avez envoyée avec une autre méthode, par exemple, **spark-submit**.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Spark sur HDInsight. Pour obtenir des instructions, consultez [Création de clusters Apache Spark dans Azure HDInsight](apache-spark-jupyter-spark-sql.md).

* Vous devriez avoir démarré le notebook, **[Machine learning : analyse prédictive des données d’inspections alimentaires à l’aide de MLLib](apache-spark-machine-learning-mllib-ipython.md)** . Pour obtenir des instructions sur l’exécution de ce bloc-notes, suivez le lien.  

## <a name="track-an-application-in-the-yarn-ui"></a>Effectuer le suivi d’une application dans l’interface utilisateur YARN

1. Lancez l’interface utilisateur YARN. Sélectionnez **Yarn** sous **Tableaux de bord du cluster**.

    ![Portail Azure - Lancer l’interface utilisateur YARN](./media/apache-spark-job-debugging/launch-apache-yarn-ui.png)

   > [!TIP]  
   > Vous pouvez également lancer l’interface utilisateur de YARN à partir de celle d’Ambari. Pour lancer l’interface utilisateur d’Ambari, sélectionnez **Accueil Ambari** sous **Tableaux de bord du cluster**. À partir de l’interface utilisateur d’Ambari, accédez à **YARN** > **Quick Links** > l’instance active de Resource Manager > **Resource Manager UI**.

2. Étant donné que vous avez démarré le travail Spark à l’aide des notebooks Jupyter, l’application porte le nom **remotesparkmagics** (nom de toutes les applications démarrées à partir des notebooks). Sélectionnez l’ID d’application en regard du nom de l’application pour obtenir plus d’informations sur le travail. Cette action lance la vue de l’application.

    ![Serveur d’historique Spark - Rechercher l’ID d’application Spark](./media/apache-spark-job-debugging/find-application-id1.png)

    Pour les applications lancées à partir des bloc-notes Jupyter, l’état est toujours **EN COURS D’EXÉCUTION** tant que vous ne fermez pas le bloc-notes.

3. Dans la vue de l’application, vous pouvez descendre pour rechercher les conteneurs associés à l’application et aux journaux d’activité (stdout/stderr). Vous pouvez également lancer l’interface utilisateur Spark en cliquant sur le lien qui correspond à l’ **URL de suivi**, comme indiqué ci-dessous.

    ![Serveur d’historique Spark - Télécharger les journaux de conteneur](./media/apache-spark-job-debugging/download-container-logs.png)

## <a name="track-an-application-in-the-spark-ui"></a>Effectuer le suivi d’une application dans l’interface utilisateur Spark

Dans l’interface utilisateur Spark, vous pouvez explorer les travaux Spark générés par l’application que vous avez démarrée précédemment.

1. Pour lancer l’interface utilisateur Spark, dans la vue de l’application, sélectionnez le lien **URL de suivi**, comme illustré dans la capture d’écran ci-dessus. Vous pouvez y voir tous les travaux Spark lancés par l’application en cours d’exécution dans le bloc-notes Jupyter.

    ![Serveur d’historique Spark - Onglet Travaux](./media/apache-spark-job-debugging/view-apache-spark-jobs.png)

2. Sélectionnez l’onglet **Exécuteurs** pour consulter les informations de traitement et de stockage pour chaque exécuteur. Vous pouvez également récupérer la pile des appels en sélectionnant le lien **Thread Dump**.

    ![Serveur d’historique Spark - Onglet Exécuteurs](./media/apache-spark-job-debugging/view-spark-executors.png)

3. Sélectionnez l’onglet **Étapes** pour consulter les étapes de l’application.

    ![Serveur d’historique Spark - Onglet Étapes](./media/apache-spark-job-debugging/view-apache-spark-stages.png "Afficher les étapes Spark")

    Chaque étape peut comporter plusieurs tâches dont vous pouvez afficher les statistiques d’exécution, comme illustré ci-dessous.

    ![Serveur d’historique Spark - Onglet Détails des étapes](./media/apache-spark-job-debugging/view-spark-stages-details.png "Afficher les détails des étapes Spark")

4. Dans la page de détails de l’étape, vous pouvez lancer la visualisation DAG. Développez le lien **DAG Visualization** (Visualisation DAG) situé en haut de la page, comme indiqué ci-dessous.

    ![Afficher la visualisation DAG des étapes Spark](./media/apache-spark-job-debugging/view-spark-stages-dag-visualization.png)

    Le graphique DAG (Direct Aclyic Graph) représente les différentes étapes de l’application. Chaque rectangle bleu dans le graphique représente une opération Spark appelée à partir de l’application.

5. Dans la page de détails de l’étape, vous pouvez également lancer la vue Chronologie de l’application. Développez le lien **Event Timeline** (Chronologie de l’événement) situé en haut de la page, comme indiqué ci-dessous.

    ![Afficher la chronologie d’événement des étapes Spark](./media/apache-spark-job-debugging/view-spark-stages-event-timeline.png)

    Cette image affiche les événements Spark sous la forme d’une chronologie. La vue chronologie est disponible sur trois niveaux : entre différents travaux, dans un travail et dans une étape. L’image ci-dessus capture la vue chronologie pour une étape donnée.

   > [!TIP]  
   > Si vous cochez la case **Activer le zoom** , vous pouvez faire défiler la vue chronologie vers la gauche et vers la droite.

6. Les autres onglets de l’interface utilisateur Spark fournissent également des informations utiles sur l’instance Spark.

   * Onglet Stockage : si votre application crée un RDD, vous trouverez des informations sous l’onglet Stockage.
   * Onglet Environnement : cet onglet fournit des informations utiles concernant votre instance Spark, par exemple :
     * version de Scala ;
     * répertoire du journal des événements associé au cluster ;
     * nombre de cœurs d’exécuteur de l’application ;

## <a name="find-information-about-completed-jobs-using-the-spark-history-server"></a>Rechercher des informations sur les tâches terminées à l’aide du serveur d’historique Spark

Une fois qu’un travail est terminé, les informations concernant ce travail sont conservées dans le serveur d’historique Spark.

1. Pour lancer le serveur d’historique Spark, dans la page **Vue d’ensemble**, sélectionnez **Serveur d’historique Spark** sous **Tableaux de bord du cluster**.

    ![Portail Azure - Lancement du serveur d’historique Spark](./media/apache-spark-job-debugging/launch-spark-history-server.png "Lancer le serveur 1 d’historique Spark")

   > [!TIP]  
   > Vous pouvez également lancer l’interface utilisateur du serveur d’historique Spark à partir de celle d’Ambari. Pour lancer l’interface utilisateur d’Ambari, depuis le volet Vue d’ensemble, sélectionnez **Accueil Ambari** sous **Tableaux de bord du cluster**. À partir de l’interface utilisateur d’Ambari, accédez à **Spark2** > **Quick Links** > **Spark2 History Server UI**.

2. Les applications terminées s’affichent dans une liste. Sélectionnez un ID d’application pour obtenir plus d’informations sur l’application.

    ![Serveur d’historique Spark - Applications terminées](./media/apache-spark-job-debugging/view-completed-applications.png "Lancer le serveur 2 d’historique Spark")

## <a name="see-also"></a>Voir aussi

* [Gérer les ressources du cluster Apache Spark dans Azure HDInsight](apache-spark-resource-manager.md)
* [Déboguer des tâches Apache Spark avec le serveur d’historique Spark étendu](apache-azure-spark-history-server.md)
* [Déboguer des applications Apache Spark avec Azure Toolkit for IntelliJ par SSH](apache-spark-intellij-tool-debug-remotely-through-ssh.md)
