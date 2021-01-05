---
title: Tutoriel - Azure Toolkit for IntelliJ (application Spark)
description: Tutoriel - Utiliser Azure Toolkit for IntelliJ pour développer des applications Spark, écrites en Scala, et les envoyer à un pool Apache Spark serverless.
services: synapse-analytics
author: hrasheed-msft
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 04/15/2020
ms.openlocfilehash: 75cc23df41741a95e87ad64d168aea4fa4f45733
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451044"
---
# <a name="tutorial-create-an-apache-spark-application-with-intellij-using-a-synapse-workspace"></a>Tutoriel : Créer une application Apache Spark avec IntelliJ en utilisant un espace de travail Synapse

Ce tutoriel vous montre comment utiliser le plug-in Azure Toolkit for IntelliJ pour développer des applications Apache Spark, qui sont écrites en [Scala](https://www.scala-lang.org/), puis les envoyer directement à un pool Apache Spark serverless à partir de l’environnement de développement intégré (IDE) IntelliJ. Vous pouvez utiliser le plug-in de différentes manières :

- Développer et soumettre une application Scala Spark sur un pool Spark.
- Accéder aux ressources de vos pools Spark.
- Développer et exécuter une application Scala Spark localement.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> - Utiliser le plug-in Azure Toolkit for IntelliJ
> - Développer des applications Apache Spark
> - Envoyer l’application aux pools Spark

## <a name="prerequisites"></a>Prérequis

- [IntelliJ IDEA version Community](https://www.jetbrains.com/idea/download/download-thanks.html?platform=windows&code=IIC).
- Plug-in du kit de ressources Azure 3.27.0-2019.2, à installer à partir du [dépôt de plug-ins IntelliJ](/java/azure/intellij/azure-toolkit-for-intellij-installation?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [JDK (version 1.8)](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).
- Plug-in Scala, à installer à partir du [dépôt de plug-ins IntelliJ](/azure/hdinsight/spark/apache-spark-intellij-tool-plugin#install-scala-plugin-for-intellij-idea).
- Le prérequis suivant concerne seulement les utilisateurs Windows :

  Quand vous exécutez l’application Spark Scala locale sur un ordinateur Windows, vous pouvez obtenir une exception, comme l’explique le document [SPARK-2356](https://issues.apache.org/jira/browse/SPARK-2356). Cette exception est liée à l’absence du fichier WinUtils.exe sur Windows.
  Pour résoudre cette erreur, [téléchargez le fichier exécutable WinUtils](http://public-repo-1.hortonworks.com/hdp-win-alpha/winutils.exe) vers un emplacement tel que **C:\WinUtils\bin**. Ajoutez ensuite la variable d’environnement **HADOOP_HOME** et définissez la valeur de la variable sur **C:\WinUtils**.

## <a name="create-a-spark-scala-application-for-a-spark-pool"></a>Créer une application Spark Scala pour un pool Spark

1. Démarrez IntelliJ IDEA, puis sélectionnez **Create New Project** (Créer un projet) pour ouvrir la fenêtre **New Project** (Nouveau projet).
2. Sélectionnez **Apache Spark/HDInsight** dans le volet gauche.
3. Sélectionnez **Spark Project avec exemples (Scala)** [Projet Spark (Scala)] dans la fenêtre principale.
4. Dans la liste déroulante **Build tool** (Outil de build), sélectionnez l’un des types suivants :

   - **Maven** pour la prise en charge de l’Assistant de création de projets Scala.
   - **SBT** pour gérer les dépendances et la génération du projet Scala.

    ![IntelliJ IDEA - Boîte de dialogue Nouveau projet](./media/intellij-tool-synapse/create-synapse-application01.png)

5. Sélectionnez **Suivant**.
6. Dans la fenêtre **New Project** (Nouveau projet), entrez les informations suivantes :

    | Propriété | Description |
    | ----- | ----- |
    |Nom du projet| Entrez un nom. Ce didacticiel utilise `myApp`.|
    |Project&nbsp;location (Emplacement du projet)| Entrez l’emplacement où vous souhaitez enregistrer votre projet.|
    |Project SDK (SDK du projet)| Ce champ peut être vide si vous utilisez IDEA pour la première fois. Sélectionnez **New...** (Nouveau) et accédez à votre JDK.|
    |Version de Spark|L’Assistant de création intègre la version correcte des SDK Spark et Scala. Synapse prend uniquement en charge **Spark 2.4.0**.|
    |||

    ![Sélection du kit de développement logiciel (SDK) Apache Spark](./media/intellij-tool-synapse/create-synapse-application02.png)

7. Sélectionnez **Terminer**. Vous devrez peut-être patienter quelques minutes avant que le projet soit disponible.
8. Le projet Spark crée automatiquement un artefact. Pour afficher l’artefact, effectuez les opérations suivantes :

   a. À partir de la barre de menus, accédez à **File** > **Project Structure**.

   b. Dans la fenêtre **Project Structure**, sélectionnez **Artifacts**.

   c. Sélectionnez **Cancel** (Annuler) après avoir visualisé l’artefact.

    ![Informations sur l’artefact dans la boîte de dialogue](./media/intellij-tool-synapse/default-artifact-dialog.png)

9. Recherchez **LogQuery** à partir de **myApp** > **src** > **main** > **scala**> **sample**> **LogQuery**. Ce tutoriel utilise **LogQuery** pour s’exécuter.

   ![Commandes pour la création d’une classe Scala à partir du projet](./media/intellij-tool-synapse/select-run-file.png)

## <a name="connect-to-your-spark-pools"></a>Vous connecter à vos pools Spark

Connectez-vous à l’abonnement Azure pour vous connecter à vos pools Spark.

### <a name="sign-in-to-your-azure-subscription"></a>Connectez-vous à votre abonnement Azure :

1. Dans la barre de menus, accédez à **Affichage** > **Fenêtres Outil** > **Azure Explorer**.

   ![IntelliJ IDEA - Affichage d’Azure Explorer](./media/intellij-tool-synapse/show-azure-explorer1.png)

2. Dans Azure Explorer, cliquez avec le bouton droit sur le nœud **Azure**, puis sélectionnez **Se connecter**.

   ![IntelliJ IDEA - Explorateur - Cliquez avec le bouton droit sur Azure](./media/intellij-tool-synapse/explorer-rightclick-azure.png)

3. Dans la boîte de dialogue **Connexion à Azure**, choisissez **Connexion à l’appareil**, puis sélectionnez **Connexion**.

    ![IntelliJ IDEA - Connexion à Azure](./media/intellij-tool-synapse/intellij-view-explorer2.png)

4. Dans la boîte de dialogue **Connexion à l’appareil Azure**, sélectionnez **Copier et ouvrir**.

   ![IntelliJ IDEA - Connexion à l’appareil Azure](./media/intellij-tool-synapse/intellij-view-explorer5.png)

5. Dans l’interface du navigateur, collez le code, puis sélectionnez **Suivant**.

   ![Microsoft - Boîte de dialogue d’entrée de code pour HDI](./media/intellij-tool-synapse/intellij-view-explorer6.png)

6. Entrez vos informations d’identification Azure, puis fermez le navigateur.

   ![Microsoft - Boîte de dialogue d’entrée de code pour HDI](./media/intellij-tool-synapse/intellij-view-explorer7.png)

7. Une fois que vous êtes connecté, la boîte de dialogue **Sélectionner des abonnements** répertorie tous les abonnements Azure associés aux informations d’identification. Sélectionnez votre abonnement, puis choisissez **Sélectionner**.

    ![Boîte de dialogue Sélectionner des abonnements](./media/intellij-tool-synapse/Select-Subscriptions.png)

8. Dans **Azure Explorer**, développez **Apache Spark on Synapse** pour voir les espaces de travail de vos abonnements.

    ![IntelliJ IDEA - Vue principale d’Azure Explorer](./media/intellij-tool-synapse/azure-explorer-workspace.png)

9. Pour voir les pools Spark, vous pouvez développer un espace de travail.

    ![Comptes de stockage Azure Explorer](./media/intellij-tool-synapse/azure-explorer-pool.png)

## <a name="remote-run-a-spark-scala-application-on-a-spark-pool"></a>Exécuter à distance une application Spark Scala sur un pool Spark

Après avoir créé une application Scala, vous pouvez l’exécuter à distance.

1. Ouvrez la fenêtre **Run/Debug Configurations** (Exécuter/déboguer les configurations) en sélectionnant l’icône.

    ![Commande d’envoi de l’application Spark à HDInsight 1](./media/intellij-tool-synapse/open-configuration-window.png)

2. Dans la boîte de dialogue **Run/Debug Configurations** (Exécuter/déboguer les configurations), sélectionnez **+** , puis sélectionnez **Apache Spark on Synapse**.

    ![Commande d’envoi de l’application Spark à HDInsight 2](./media/intellij-tool-synapse/create-synapse-configuration02.png)

3. Dans la fenêtre **Run/Debug Configurations** (Exécuter/déboguer les configurations), entrez les valeurs suivantes, puis sélectionnez **OK** :

    |Propriété |Valeur |
    |----|----|
    |Spark pools|Sélectionnez les pools Spark sur lesquels vous souhaitez exécuter votre application.|
    |Sélectionner un artefact à envoyer|Conservez le paramètre par défaut.|
    |Main class name|La valeur par défaut est la classe principale du fichier sélectionné. Vous pouvez changer la classe en sélectionnant les points de suspension ( **...** ), puis en choisissant une autre classe.|
    |Configurations du travail|Vous pouvez changer la clé et les valeurs par défaut. Pour plus d’informations, consultez [API REST Apache Livy](http://livy.incubator.apache.org./docs/latest/rest-api.html).|
    |Arguments de ligne de commande|Vous pouvez entrer des arguments séparés par un espace pour la classe main, si nécessaire.|
    |JAR référencés et fichiers référencés|vous pouvez entrer les chemins des fichiers jar et des fichiers référencés, si vous en avez. Vous pouvez également parcourir les fichiers dans le système de fichiers virtuel Azure, qui ne prend en charge que le cluster ADLS Gen2. Pour plus d’informations, consultez [Configuration Apache Spark]https://spark.apache.org/docs/2.4.5/configuration.html#runtime-environment) et [Guide pratique pour charger des ressources sur un cluster](../../storage/blobs/storage-quickstart-blobs-storage-explorer.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).|
    |Stockage des chargements de travaux|Développez pour afficher des options supplémentaires.|
    |Type de stockage|Sélectionnez **Utiliser l’objet blob Azure pour charger** ou **Utiliser le compte de stockage par défaut du cluster pour charger** à partir de la liste déroulante.|
    |Compte de stockage|Entrez votre compte de stockage.|
    |Clé de stockage|Entrez votre clé de stockage.|
    |Conteneur de stockage|Sélectionnez votre conteneur de stockage dans la liste déroulante après avoir entré les valeurs **Compte de stockage** et **Clé de stockage**.|

    ![Boîte de dialogue Spark Submission (Envoi Spark) 1](./media/intellij-tool-synapse/create-synapse-configuration03.png)

4. Sélectionnez l’icône **SparkJobRun** pour envoyer votre projet au pool Spark sélectionné. L’onglet **Remote Spark Job in Cluster** (Travail Spark distant dans le cluster) affiche la progression de l’exécution du travail au bas de la page. Vous pouvez arrêter l’application en sélectionnant le bouton rouge.

    ![Fenêtre Apache Spark Submission (Envoi Spark)](./media/intellij-tool-synapse/remotely-run-synapse.png)

    ![Boîte de dialogue Spark Submission (Envoi Spark) 2](./media/intellij-tool-synapse/remotely-run-result.png)

## <a name="local-rundebug-apache-spark-applications"></a>Exécuter et déboguer les applications Apache Spark localement

Vous pouvez suivre les instructions ci-dessous pour configurer l’exécution et le débogage locaux de votre travail Apache Spark.

### <a name="scenario-1-do-local-run"></a>Scénario 1 : Effectuer une exécution locale

1. Ouvrez la boîte de dialogue **Run/Debug Configurations** (Exécuter/Déboguer les configurations), puis sélectionnez le signe plus ( **+** ). Sélectionnez ensuite l’option **Apache Spark on Synapse**. Entrez les informations sur le nom (**Name**) et le nom principal de la classe (**Main class name**) à enregistrer.

    ![Intellij - Exécution locale des configuration de débogage 1](./media/intellij-tool-synapse/local-run-synapse.png)

    - Les variables d’environnement et l’emplacement de WinUtils.exe ne concernent que les utilisateurs Windows.
    - Variables d’environnement : la variable d’environnement système peut être détectée automatiquement si vous l’avez définie avant et que vous n’avez pas besoin de l’ajouter manuellement.
    - [Emplacement de WinUtils.exe](http://public-repo-1.hortonworks.com/hdp-win-alpha/winutils.exe) : Vous pouvez spécifier l’emplacement de WinUtils en sélectionnant l’icône de dossier à droite.

2. Sélectionnez ensuite le bouton de lecture locale.

    ![Intellij - Exécution locale des configuration de débogage 2](./media/intellij-tool-synapse/local-run-synapse01.png)

3. Une fois l’exécution locale terminée, si le script contient une sortie, vous pouvez vérifier le fichier de sortie à partir de **data** > **__default__**.

    ![Intellij - Résultat de l’exécution locale du projet 1](./media/intellij-tool-synapse/spark-local-run-result.png)

### <a name="scenario-2-do-local-debugging"></a>Scénario 2 : Effectuer un débogage local

1. Ouvrez le script **LogQuery**, puis définissez des points d’arrêt.
2. Sélectionnez l’icône **Local debug** (Débogage local) pour effectuer le débogage local.

    ![Intellij - Résultat d’exécution locale du projet 2](./media/intellij-tool-synapse/local-debug-synapse.png)

## <a name="access-and-manage-synapse-workspace"></a>Accéder à l’espace de travail Synapse et le gérer

Vous pouvez effectuer différentes opérations dans Azure Explorer au sein d’Azure Toolkit for IntelliJ. Dans la barre de menus, accédez à **Affichage** > **Fenêtres Outil** > **Azure Explorer**.

### <a name="launch-workspace"></a>Lancer l’espace de travail

1. Dans Azure Explorer, accédez à **Apache Spark on Synapse**, puis développez-le.

    ![IntelliJ IDEA - Vue principale d’Azure Explorer](./media/intellij-tool-synapse/azure-explorer-workspace.png)

2. Cliquez avec le bouton droit sur un espace de travail, puis sélectionnez **Launch workspace** (Lancer l’espace de travail) ; le site web s’ouvre.

    ![Travail Spark - Affichage des détails de l’application 1](./media/intellij-tool-synapse/launch-workspace-synapse.png)

    ![Travail Spark - Affichage des détails de l’application 2](./media/intellij-tool-synapse/launch-workspace-result.png)

## <a name="spark-console"></a>Console Spark

Vous pouvez exécuter la console locale Spark (Scala) ou exécuter la console de sessions interactives Spark Livy (Scala).

### <a name="spark-local-console-scala"></a>Console locale Spark (Scala)

Veillez à respecter les prérequis WINUTILS.EXE.

1. Dans la barre de menus, accédez à **Run** > **Edit Configurations...** .
2. Dans la fenêtre **Run/Debug Configurations** (Exécuter/déboguer des configurations), dans le volet gauche, accédez à **Apache Spark on Synapse** >  **[Spark on Synapse] myApp**.
3. Dans la fenêtre principale, sélectionnez l’onglet **Locally Run**.
4. Entrez les valeurs suivantes, puis sélectionnez **OK** :

    |Propriété |Valeur |
    |----|----|
    |Variables d'environnement|Vérifiez que la valeur de HADOOP_HOME est correcte.|
    |WINUTILS.exe location|Vérifiez que le chemin est correct.|

    ![Configuration de la console locale](./media/intellij-tool-synapse/local-console-synapse01.png)

5. Dans le projet, accédez à **myApp** > **src** > **main** > **scala** > **myApp**.
6. Dans la barre de menus, accédez à **Tools** > **Spark console** > **Run Spark Local Console(Scala)** .
7. Deux boîtes de dialogue peuvent s’afficher pour vous demander si vous souhaitez corriger automatiquement les dépendances. Si c’est le cas, sélectionnez **Correction automatique**.

    ![IntelliJ IDEA Spark - Boîte de dialogue1 Auto Fix](./media/intellij-tool-synapse/intellij-console-autofix1.png)

    ![IntelliJ IDEA Spark - Boîte de dialogue2 Auto Fix](./media/intellij-tool-synapse/intellij-console-autofix2.png)

8. La console doit ressembler à l’image ci-dessous. Dans la fenêtre de la console, tapez `sc.appName`, puis appuyez sur Ctrl + Entrée. Le résultat s’affiche. Vous pouvez arrêter l’exécution de la console locale en sélectionnant le bouton rouge.

    ![IntelliJ IDEA - Résultat de la console locale](./media/intellij-tool-synapse/local-console-result.png)

### <a name="spark-livy-interactive-session-console-scala"></a>Console de sessions interactives Spark Livy (Scala)

Elle est uniquement prise en charge sur IntelliJ 2018.2 et 2018.3.

1. Dans la barre de menus, accédez à **Run** > **Edit Configurations...** .

2. Dans la fenêtre **Run/Debug Configurations** (Exécuter/déboguer des configurations), dans le volet gauche, accédez à **Apache Spark on Synapse** >  **[Spark on Synapse] myApp**.

3. Dans la fenêtre principale, sélectionnez l’onglet **Remotely Run in Cluster**.

4. Entrez les valeurs suivantes, puis sélectionnez **OK** :

    |Propriété |Valeur |
    |----|----|
    |Main class name| Sélectionnez le nom de la classe Main.| 
    |Spark pools|Sélectionnez les pools Spark sur lesquels vous souhaitez exécuter votre application.|
    ||

    ![Configuration de la console interactive](./media/intellij-tool-synapse/interactive-console-configuration.png)

5. Dans le projet, accédez à **myApp** > **src** > **main** > **scala** > **myApp**.

6. Dans la barre de menus, accédez à **Tools** > **Spark console** > **Run Spark Livy Interactive Session Console(Scala)** .
7. La console doit ressembler à l’image ci-dessous. Dans la fenêtre de la console, tapez `sc.appName`, puis appuyez sur Ctrl + Entrée. Le résultat s’affiche. Vous pouvez arrêter l’exécution de la console locale en sélectionnant le bouton rouge.

    ![IntelliJ IDEA - Résultat de la console interactive](./media/intellij-tool-synapse/interactive-console-result.png)

### <a name="send-selection-to-spark-console"></a>Envoyer la sélection vers la console Spark

Vous pouvez voir le résultat du script en envoyant du code vers la console locale ou la console de sessions interactives Livy (Scala). Pour cela, vous pouvez mettre en surbrillance du code dans le fichier Scala, puis cliquer avec le bouton droit sur **Send Selection To Spark Console** (Envoyer la sélection vers la console Spark). Le code sélectionné est envoyé vers la console pour être exécuté. Le résultat s’affiche après le code dans la console. La console va vérifie les éventuelles erreurs.

   ![Envoyer la sélection vers la console Spark](./media/intellij-tool-synapse/send-selection-to-console.png)

## <a name="next-steps"></a>Étapes suivantes

- [Azure Synapse Analytics](../overview-what-is.md)
- [Créer un pool Apache Spark pour un espace de travail Azure Synapse Analytics](../../synapse-analytics/quickstart-create-apache-spark-pool-studio.md)
