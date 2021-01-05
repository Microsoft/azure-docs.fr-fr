---
title: 'Tutoriel : Application Scala Maven pour Spark et IntelliJ – Azure HDInsight'
description: 'Tutoriel : Créer une application Spark écrite en Scala avec Apache Maven comme système de génération. Et un archétype Maven existant pour Scala fourni par IntelliJ IDEA.'
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: contperf-fy21q1
ms.date: 08/21/2020
ms.openlocfilehash: 5be4ce97e6087847f084f9b9ec2a6b8b0df84e14
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033508"
---
# <a name="tutorial-create-a-scala-maven-application-for-apache-spark-in-hdinsight-using-intellij"></a>Tutoriel : Créer une application Scala Maven pour Apache Spark dans HDInsight à l’aide d’IntelliJ

Dans ce tutoriel, vous allez apprendre à créer une application Apache Spark écrite en Scala à l’aide d’Apache Maven avec IntelliJ IDEA. Dans cet article, Apache Maven est utilisé comme système de génération. Le document présente dans un premier temps un archétype Maven existant pour Scala fourni par IntelliJ IDEA.  La création d’une application Scala avec IntelliJ IDEA implique les étapes suivantes :

* Utiliser Maven en tant que le système de génération.
* Mettre à jour le fichier de modèle objet du projet (POM) pour résoudre les dépendances de module Spark.
* Écrire votre application dans Scala.
* Générer un fichier jar qui peut être soumis à des clusters HDInsight Spark.
* Exécuter l’application à distance sur le cluster Spark à l’aide de Livy.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Installer le plug-in Scala pour IntelliJ IDEA
> * Utiliser IntelliJ pour développer une application Scala Maven
> * Créer un programme Scala autonome

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Spark sur HDInsight. Pour obtenir des instructions, consultez [Création de clusters Apache Spark dans Azure HDInsight](apache-spark-jupyter-spark-sql.md).

* Le [SDK Oracle Java](https://www.azul.com/downloads/azure-only/zulu/).  Ce tutoriel utilise Java version 8.0.202.

* IDE Java. Cet article utilise [IntelliJ IDEA Community  version 2018.3.4](https://www.jetbrains.com/idea/download/).

* Azure Toolkit for IntelliJ.  Consultez [Installation d’Azure Toolkit for IntelliJ](/java/azure/intellij/azure-toolkit-for-intellij-create-hello-world-web-app#installation-and-sign-in).

## <a name="install-scala-plugin-for-intellij-idea"></a>Installer le plug-in Scala pour IntelliJ IDEA

Effectuez les étapes suivantes pour installer le plug-in Scala :

1. Ouvrez IntelliJ IDEA.

2. Dans l’écran d’accueil, accédez à **Configure** > **Plugins** (Configurer > Plug-ins) pour ouvrir la fenêtre **Plugins** (Plug-ins).

    ![« IntelliJ IDEA - Activation du plug-in Scala »](./media/apache-spark-create-standalone-application/enable-scala-plugin1.png)

3. Sélectionnez **Install** (Installer) pour le plug-in Scala proposé dans la nouvelle fenêtre.  

    ![« IntelliJ IDEA - Installation du plug-in Scala »](./media/apache-spark-create-standalone-application/install-scala-plugin.png)

4. Une fois que le plug-in est bien installé, vous devez redémarrer l’IDE.

## <a name="use-intellij-to-create-application"></a>Utilisez IntelliJ pour créer l’application

1. Démarrez IntelliJ IDEA, puis sélectionnez **Create New Project** (Créer un projet) pour ouvrir la fenêtre **New Project** (Nouveau projet).

2. Sélectionnez **Apache Spark/HDInsight** dans le volet gauche.

3. Sélectionnez **Spark Project (Scala)** [Projet Spark (Scala)] dans la fenêtre principale.

4. Dans la liste déroulante **Build tool** (Outil de build), sélectionnez une des valeurs suivantes :
      * **Maven** pour la prise en charge de l’Assistant de création de projets Scala.
      * **SBT** pour gérer les dépendances et la génération du projet Scala.

   ![IntelliJ - Boîte de dialogue Nouveau projet](./media/apache-spark-create-standalone-application/intellij-project-apache-spark.png)

5. Sélectionnez **Suivant**.

6. Dans la fenêtre **New Project** (Nouveau projet), entrez les informations suivantes :  

  	|  Propriété   | Description   |  
  	| ----- | ----- |  
  	|Nom du projet| Entrez un nom.|  
  	|Project&nbsp;location (Emplacement du projet)| Entrez l’emplacement où enregistrer votre projet.|
  	|Project SDK (SDK du projet)| Ce champ est vide si vous utilisez IDEA pour la première fois.  Sélectionnez **New...** (Nouveau) et accédez à votre JDK.|
  	|Version de Spark|L’Assistant de création intègre la version correcte des SDK Spark et Scala. Si la version du cluster Spark est antérieure à la version 2.0, sélectionnez **Spark 1.x**. Sinon, sélectionnez **Spark 2.x**. Cet exemple utilise **Spark 2.3.0 (Scala 2.11.8)** .|

    ![IntelliJ IDEA - Sélection du SDK Spark](./media/apache-spark-create-standalone-application/hdi-scala-new-project.png)

7. Sélectionnez **Terminer**.

## <a name="create-a-standalone-scala-project"></a>Créer un programme Scala autonome

1. Démarrez IntelliJ IDEA, puis sélectionnez **Create New Project** (Créer un projet) pour ouvrir la fenêtre **New Project** (Nouveau projet).

2. Sélectionnez **Maven** dans le volet gauche.

3. Spécifiez un **projet SDK**. Si ce champ est vide, sélectionnez **New...** (Nouveau), puis accédez au répertoire d’installation de Java.

4. Cochez la case **Create from archetype** (Créer à partir d’un archétype).  

5. Dans la liste des archétypes, sélectionnez **`org.scala-tools.archetypes:scala-archetype-simple`** . Cet archétype crée la structure de répertoire appropriée et télécharge les dépendances par défaut requises pour écrire le programme Scala.

    ![Capture d’écran montrant l’archétype sélectionnés dans la fenêtre New Project (Nouveau projet).](./media/apache-spark-create-standalone-application/intellij-project-create-maven.png)

6. Sélectionnez **Suivant**.

7. Développez **Artifact Coordinates** (Coordonnées d’artefact). Spécifiez les valeurs appropriées pour **GroupId** et **ArtifactId**. **Nom** et **Emplacement** sont renseignés automatiquement. Les valeurs suivantes sont utilisées dans ce didacticiel :

    - **GroupId** : com.microsoft.spark.example
    - **ArtifactId** : SparkSimpleApp

    ![Capture d’écran montrant l’option Artifact Coordinates (Coordonnées d’artefact) dans la fenêtre New Project (Nouveau projet).](./media/apache-spark-create-standalone-application/intellij-artifact-coordinates.png)

8. Sélectionnez **Suivant**.

9. Vérifiez les paramètres, puis sélectionnez **Suivant**.

10. Vérifiez le nom et l’emplacement du projet, puis sélectionnez **Terminer**.  L’importation du projet prend quelques minutes.

11. Une fois que le projet a été importé, à partir du volet gauche, accédez à **SparkSimpleApp** > **src** > **test** > **scala** > **com** > **microsoft** > **spark** > **example**.  Cliquez avec le bouton droit sur **MySpec**, puis sélectionnez **Delete...** (Supprimer). Vous n’avez pas besoin de ce fichier pour l’application.  Sélectionnez **OK** dans la boîte de dialogue.
  
12. Dans les étapes ultérieures, vous mettez à jour le fichier **pom.xml** pour définir les dépendances de l’application Spark Scala. Pour que ces dépendances soient téléchargées et résolues automatiquement, vous devez configurer Maven.

13. Dans le menu **File** (Fichier), sélectionnez **Settings** (Paramètres) pour ouvrir la fenêtre **Settings** (Paramètres).

14. Dans la fenêtre **Settings** (Paramètres), accédez à **Build, Execution, Deployment** > **Build Tools** > **Maven** > **Importing** (Build, Exécution, Déploiement > Outils de build > Maven >Importation).

15. Cochez la case **Import Maven projects automatically** (Importer les projets Maven automatiquement).

16. Sélectionnez **Apply** (Appliquer), puis **OK**.  Vous revenez à la fenêtre du projet.

    ![Configurer Maven pour les téléchargements automatiques](./media/apache-spark-create-standalone-application/configure-maven-download.png)

17. À partir du volet gauche, accédez à **src** > **main** > **scala** > **com.microsoft.spark.example**, puis double-cliquez sur **App** pour ouvrir App.scala.

18. Remplacez le code existant par le code suivant et enregistrez les modifications. Ce code lit les données du fichier HVAC.csv (disponible sur tous les clusters HDInsight Spark). Récupère les lignes qui contiennent uniquement un chiffre dans la sixième colonne et écrit la sortie dans **/HVACOut** sous le conteneur de stockage par défaut du cluster.

    ```scala
    package com.microsoft.spark.example

    import org.apache.spark.SparkConf
    import org.apache.spark.SparkContext

    /**
      * Test IO to wasb
      */
    object WasbIOTest {
        def main (arg: Array[String]): Unit = {
            val conf = new SparkConf().setAppName("WASBIOTest")
            val sc = new SparkContext(conf)
    
            val rdd = sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    
            //find the rows which have only one digit in the 7th column in the CSV
            val rdd1 = rdd.filter(s => s.split(",")(6).length() == 1)
    
            rdd1.saveAsTextFile("wasb:///HVACout")
        }
    }
    ```

19. Dans le volet gauche, double-cliquez sur **pom.xml**.  

20. Ajoutez les segments suivants à `<project>\<properties>` :

    ```xml
    <scala.version>2.11.8</scala.version>
    <scala.compat.version>2.11.8</scala.compat.version>
    <scala.binary.version>2.11</scala.binary.version>
    ```

21. Ajoutez les segments suivants à `<project>\<dependencies>` :

    ```xml
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_${scala.binary.version}</artifactId>
        <version>2.3.0</version>
    </dependency>
    ```

    Enregistrez les modifications apportées au fichier pom.xml.

22. Créez le fichier .jar IntelliJ IDEA permet de créer le fichier JAR en tant qu’artefact d'un projet. Procédez comme suit.

    1. Dans le menu **File** (Fichier), sélectionnez **Project Structure...** (Structure du projet).

    2. À partir de la fenêtre **Project Structure** (Structure du projet), accédez à **Artifacts** > **le signe +**  > **JAR** > **From modules with dependencies...** (À partir de modules avec des dépendances).

        ![« IntelliJ IDEA - Ajout d’un fichier jar dans la structure du projet »](./media/apache-spark-create-standalone-application/hdinsight-create-jar1.png)

    3. Dans la fenêtre **Create JAR from Modules** (Créer un fichier JAR à partir de modules), sélectionnez l’icône de dossier dans la zone de texte **Main Class** (Classe principale).

    4. Dans la fenêtre **Select Main Class** (Sélectionner une classe principale), sélectionnez la classe qui s’affiche par défaut, puis sélectionnez **OK**.

        ![« IntelliJ IDEA - Sélection d’une classe dans la structure du projet »](./media/apache-spark-create-standalone-application/hdinsight-create-jar2.png)

    5. Dans la fenêtre **Create JAR from Modules** (Créer un fichier JAR à partir de modules), assurez-vous que l’option **Extract to the target JAR** (Extraire vers le fichier JAR cible) est sélectionnée, puis sélectionnez **OK**.  Ce paramètre crée un fichier JAR unique contenant toutes les dépendances.

        ![IntelliJ IDEA - Fichier jar de la structure du projet dans le module](./media/apache-spark-create-standalone-application/hdinsight-create-jar3.png)

    6. L’onglet **Output Layout** (Disposition de la sortie) répertorie tous les fichiers jar inclus dans le cadre du projet Maven. Vous pouvez sélectionner et supprimer ceux sur lesquels l’application Scala n’a aucune dépendance directe. Pour l’application que vous créez ici, vous pouvez tous les supprimer sauf le dernier (**SparkSimpleApp compile output**). Sélectionnez les fichiers JAR à supprimer, puis sélectionnez le signe moins **-** .

        ![« IntelliJ IDEA - Suppression de la sortie dans la structure du projet »](./media/apache-spark-create-standalone-application/hdi-delete-output-jars.png)

        Vérifiez que la case **Include in project build** (Inclure dans la build du projet) est cochée. Cette option garantit la création du fichier jar lors de chaque génération ou mise à jour du projet. Sélectionnez **Apply** (Appliquer), puis **OK**.

    7. Pour créer le fichier JAR, accédez à **Build** > **Build Artifacts** > **Build** (Build > Générer les artefacts > Générer). Le projet est compilé en 30 secondes environ.  Le fichier JAR de sortie est créé sous **\out\artifacts**.

        ![IntelliJ IDEA - Sortie d’artefact du projet](./media/apache-spark-create-standalone-application/hdi-artifact-output-jar.png)

## <a name="run-the-application-on-the-apache-spark-cluster"></a>Exécuter l’application sur le cluster Apache Spark

Pour exécuter l’application sur le cluster, vous pouvez procéder comme suit :

* **Copiez le fichier jar de l’application sur l’objet blob de Stockage Azure** associé au cluster. Pour ce faire, vous pouvez utiliser l’utilitaire en ligne de commande **AzCopy**. De nombreux autres clients permettent également de charger des données. Pour en savoir plus à leur sujet, consultez [Chargement de données pour les tâches Apache Hadoop dans HDInsight](../hdinsight-upload-data.md).

* **Utilisez Apache Livy pour soumettre une tâche d’application à distance** au cluster Spark. Les clusters Spark sur HDInsight incluent Livy qui expose des points de terminaison REST permettant d’envoyer à distance des travaux Spark. Pour plus d’informations, consultez [Envoi de tâches Apache Spark à distance en utilisant Apache Livy avec les clusters Spark sur HDInsight](apache-spark-livy-rest-interface.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, effectuez les étapes suivantes pour supprimer le cluster que vous avez créé :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Dans la zone **Recherche** située en haut, tapez **HDInsight**.

1. Sous **Services**, sélectionnez **Clusters HDInsight**.

1. Dans la liste des clusters HDInsight qui s’affiche, sélectionnez les points de suspension **...** à côté du cluster que vous avez créé pour ce tutoriel.

1. Sélectionnez **Supprimer**. Sélectionnez **Oui**.

![« Supprimer le cluster HDInsight dans le portail Azure »](./media/apache-spark-create-standalone-application/hdinsight-azure-portal-delete-cluster.png "Supprimer un cluster HDInsight")

## <a name="next-step"></a>Étape suivante

Dans cet article, vous avez appris à créer une application Apache Spark Scala. Passez à l’article suivant pour savoir comment exécuter cette application sur un cluster HDInsight Spark en utilisant Livy.

> [!div class="nextstepaction"]
>[Exécuter des tâches à distance avec Apache Livy sur un cluster Apache Spark](./apache-spark-livy-rest-interface.md)