---
title: Azure HDInsight pour Visual Studio Code
description: Découvrez comment utiliser les outils Spark et Hive (Azure HDInsight) pour Visual Studio Code. Utilisez les outils pour créer et envoyer des requêtes et des scripts.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 10/20/2020
ms.custom: devx-track-python
ms.openlocfilehash: 302f1a081ca44cf6436f2c318b03e227f6640489
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001964"
---
# <a name="use-spark--hive-tools-for-visual-studio-code"></a>Utilisez les outils Spark et Hive pour Visual Studio Code

Découvrez comment utiliser les outils Spark et Hive Apache pour Visual Studio Code. Utilisez les outils pour créer et envoyer des travaux Apache Hive de traitement par lots, des requêtes Hive interactives et des scripts PySpark pour Apache Spark. Tout d’abord, nous décrirons comment installer les outils Spark et Hive dans Visual Studio Code. Ensuite, nous verrons comment envoyer des travaux aux outils Spark et Hive.  

Les outils Spark et Hive peuvent être installée sur les plateformes prises en charge par Visual Studio Code. Les différentes plateformes ont les prérequis suivants.

## <a name="prerequisites"></a>Prérequis

Les éléments suivants sont requis pour effectuer les étapes décrites dans cet article :

- Un cluster Azure HDInsight. Pour créer un cluster, consultez [Prise en main de HDInsight](hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md). Vous pouvez également utiliser un cluster Spark et Hive qui prend en charge un point de terminaison Apache Livy.
- [Visual Studio Code](https://code.visualstudio.com/).
- [Mono](https://www.mono-project.com/docs/getting-started/install/). Mono est nécessaire uniquement pour les plateformes Linux et MacOS.
- [Un environnement interactif PySpark pour Visual Studio Code](set-up-pyspark-interactive-environment.md).
- Un répertoire local. Cet article utilise **C:\HD\HDexample**.

## <a name="install-spark--hive-tools"></a>Installer l’extension Spark & Hive Tools

Une fois que vous disposez de tous les éléments prérequis, vous pouvez installer les outils Spark et Hive pour Visual Studio Code en procédant comme suit :

1. Ouvrez Visual Studio Code.

2. Dans la barre de menus, accédez à **Afficher** > **Extensions**.

3. Dans la zone de recherche, entrez **Spark & Hive**.

4. Sélectionnez **Outils Spark et Hive** dans les résultats de la recherche, puis sélectionnez **Installer** :

   ![Installation de Python avec Spark et Hive pour Visual Studio Code](./media/hdinsight-for-vscode/install-hdInsight-plugin.png)

5. Sélectionnez **Recharger** si nécessaire.

## <a name="open-a-work-folder"></a>Ouvrir un dossier de travail

Pour ouvrir un dossier de travail et créer un fichier dans Visual Studio Code, procédez comme suit :

1. À partir de la barre de menus, accédez à **Fichier** > **Ouvrir le dossier...**  > **C:\HD\HDexample**, puis sélectionnez le bouton **Sélectionner le dossier**. Le dossier s’affiche dans l’affichage **Explorateur** sur la gauche.

2. Dans la vue **Explorateur**, sélectionnez le dossier **HDexample**, puis l’icône **Nouveau fichier** à côté du dossier de travail :

   ![icône nouveau fichier visual studio code](./media/hdinsight-for-vscode/visual-studio-code-new-file.png)

3. Nommez le nouveau fichier avec l’extension de fichier `.hql` (requêtes Hive) ou `.py` (script Spark). Cet exemple utilise **HelloWorld.hql**.

## <a name="set-the-azure-environment"></a>Configurer l’environnement Azure

Pour un utilisateur de cloud national, suivez ces étapes pour configurer l’environnement Azure, puis utilisez la commande **Azure: Sign In** pour vous connecter à Azure :

1. Accédez à **Fichier** > **Préférences** > **Paramètres**.
2. Recherchez la chaîne suivante : **Azure : Cloud**.
3. Sélectionnez le cloud national dans la liste :

   ![Définir la configuration de l’entrée de connexion par défaut](./media/hdinsight-for-vscode/set-default-login-entry-configuration.png)

## <a name="connect-to-an-azure-account"></a>Se connecter à un compte Azure

Pour que vous puissiez envoyer des scripts à vos clusters à partir de Visual Studio Code, l’utilisateur a la possibilité de se connecter à un abonnement Azure ou de [lier un cluster HDInsight](#link-a-cluster). Utilisez le nom d’utilisateur/mot de passe Ambari ou les informations d’identification jointes au domaine de votre cluster ESP pour vous connecter à votre cluster HDInsight. Suivez ces étapes pour vous connecter à Azure :

1. À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes...** , puis entrez **Azure : Sign In** :

   ![Outils Spark et Hive pour la connexion à Visual Studio Code](./media/hdinsight-for-vscode/hdinsight-for-vscode-extension-login.png)

2. Suivez les instructions de connexion pour vous connecter à Azure. Une fois que vous êtes connecté, le nom de votre compte Azure apparaît dans la barre d’état, en bas de la fenêtre Visual Studio Code.  

## <a name="link-a-cluster"></a>Lier un cluster

### <a name="link-azure-hdinsight"></a>Lien : Azure HDInsight

Vous pouvez lier un cluster normal à l’aide d’un nom d’utilisateur [Apache Ambari](https://ambari.apache.org/) managé ou lier un cluster de sécurité Hadoop de Pack Sécurité Entreprise à l’aide d’un nom d’utilisateur de domaine (par exemple : `user1@contoso.com`).

1. À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes...** , puis entrez **Spark/Hive : Link a Cluster** (Spark / Hive : lier un cluster).

   ![Commande de cluster Lien Palette de commandes](./media/hdinsight-for-vscode/link-cluster-command.png)

2. Sélectionnez le type de cluster lié **Azure HDInsight**.

3. Entrez l’URL du cluster HDInsight.

4. Entrez votre nom d’utilisateur Ambari (**admin** par défaut).

5. Entrez votre mot de passe Ambari.

6. Sélectionnez le type de cluster.

7. Définissez le nom d’affichage du cluster (facultatif).

8. Examinez l’affichage **OUTPUT** (SORTIE).

   > [!NOTE]  
   > Le nom d’utilisateur lié et le mot de passe sont utilisés si le cluster est à la fois connecté à l’abonnement Azure et lié à un cluster.  

### <a name="link-generic-livy-endpoint"></a>Lien : point de terminaison Livy générique

1. À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes...** , puis entrez **Spark/Hive : Link a Cluster** (Spark / Hive : lier un cluster).

2. Sélectionnez le type de cluster lié **Generic Livy Endpoint**.

3. Entrez le point de terminaison Livy générique. Par exemple, http\://10.172.41.42:18080.

4. Sélectionnez le type d’autorisation **De base** ou **Aucune**.  Si vous sélectionnez **De base** :  
   
   1. Entrez votre nom d’utilisateur Ambari (**admin** par défaut).  

   2. Entrez votre mot de passe Ambari.

5. Examinez l’affichage **OUTPUT** (SORTIE).

## <a name="list-clusters"></a>Lister les clusters

1. À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes...** , puis entrez **Spark/Hive : List Cluster**.

2. Sélectionnez l’abonnement souhaité.

3. Examinez l’affichage **OUTPUT** (SORTIE). Cet affichage montre le ou les clusters liés et tous les clusters de votre abonnement Azure :

   ![Définir une configuration de cluster par défaut](./media/hdinsight-for-vscode/list-cluster-result1.png)

## <a name="set-the-default-cluster"></a>Définir le cluster par défaut

1. Rouvrez le dossier **HDexample** évoqué [précédemment](#open-a-work-folder) s’il est fermé.  

2. Sélectionnez le fichier **HelloWorld.hql** créé [précédemment](#open-a-work-folder). Il s’ouvre dans l’éditeur de script.

3. Cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez **Spark / Hive: Set Default Cluster** (Spark / Hive : définir le cluster par défaut).  

4. [Connectez-vous](#connect-to-an-azure-account) à votre compte Azure ou liez un cluster si ce n’est déjà fait.

5. Sélectionnez un cluster comme cluster par défaut pour le fichier de script actuel. Les outils mettent automatiquement à jour le fichier de configuration **.VSCode\settings.json** :

   ![Définir une configuration de cluster par défaut](./media/hdinsight-for-vscode/set-default-cluster-configuration.png)

## <a name="submit-interactive-hive-queries-and-hive-batch-scripts"></a>Envoyer des requêtes Hive interactives et des scripts de commandes par lot Hive

Avec les outils Spark et Hive pour Visual Studio Code, vous pouvez envoyer des requêtes Hive interactives et des scripts de commandes par lot Hive à vos clusters.

1. Rouvrez le dossier **HDexample** évoqué [précédemment](#open-a-work-folder) s’il est fermé.  

2. Sélectionnez le fichier **HelloWorld.hql** créé [précédemment](#open-a-work-folder). Il s’ouvre dans l’éditeur de script.

3. Copiez le code suivant et collez-le dans votre fichier Hive, puis enregistrez-le :

   ```hiveql
   SELECT * FROM hivesampletable;
   ```

4. [Connectez-vous](#connect-to-an-azure-account) à votre compte Azure ou liez un cluster si ce n’est déjà fait.

5. Cliquez avec le bouton droit sur l’éditeur de script et sélectionnez **Hive: Interactive** pour envoyer la requête, ou utilisez le raccourci clavier Ctrl+Alt+I.  Sélectionnez **Hive : Batch** pour envoyer le script, ou utilisez le raccourci clavier Ctrl+Alt+H.  

6. Si vous n’avez pas spécifié de cluster par défaut, sélectionnez un cluster. Les outils vous permettent également d’envoyer un bloc de code au lieu du fichier de script entier à partir du menu contextuel. Peu après, les résultats de la requête s’affichent dans un nouvel onglet :

   ![Résultat de requête interactive Apache Hive](./media/hdinsight-for-vscode/interactive-hive-result.png)

   - Volet **RÉSULTATS** : Vous pouvez enregistrer le résultat complet dans un fichier CSV, JSON ou Excel dans un chemin local ou enregistrer seulement certaines lignes du résultat.

   - Volet **MESSAGES** : Cliquez sur un numéro de **ligne** pour accéder à la première ligne du script en cours d’exécution.

## <a name="submit-interactive-pyspark-queries"></a>Envoyer des requêtes PySpark interactives

Les utilisateurs peuvent exécuter PySpark de manière interactive de plusieurs façons :

### <a name="using-the-pyspark-interactive-command-in-py-file"></a>Commande interactive PySpark dans le fichier PY
En utilisant la commande interactive PySpark pour envoyer les requêtes, suivez la procédure ci-dessous :

1. Rouvrez le dossier **HDexample** évoqué [précédemment](#open-a-work-folder) s’il est fermé.  

2. Créez un fichier **HelloWorld.py** en suivant les étapes [précédentes](#open-a-work-folder).

3. Copiez le code suivant et collez-le dans le fichier de script :

   ```python
   from operator import add
   lines = spark.read.text("/HdiSamples/HdiSamples/FoodInspectionData/README").rdd.map(lambda r: r[0])
   counters = lines.flatMap(lambda x: x.split(' ')) \
                .map(lambda x: (x, 1)) \
                .reduceByKey(add)

   coll = counters.collect()
   sortedCollection = sorted(coll, key = lambda r: r[1], reverse = True)

   for i in range(0, 5):
        print(sortedCollection[i])
   ```

4. L’invite d’installation du noyau PySpark/Synapse Pyspark s’affiche en bas à droite de la fenêtre. Vous pouvez cliquer sur le bouton **Installer** pour continuer l’installation de PySpark/Synapse Pyspark, ou sur le bouton **Ignorer** pour ignorer cette étape.

   ![Capture d’écran montrant une option permettant d’ignorer l’installation de PySpark.](./media/hdinsight-for-vscode/install-the-pyspark-kernel.png)

5. Si vous avez besoin de l’installer ultérieurement, vous pouvez accéder à **Fichier** > **Préférence** > **Paramètres**, puis décocher **HDInsight : Activer Ignorer l’installation de PySpark** dans les paramètres. 
    
    ![Capture d’écran montrant l’option permettant d’activer l’installation de Pyspark.](./media/hdinsight-for-vscode/enable-skip-pyspark-installation.png)

6. Si l’installation réussit à l’étape 4, la zone de message « PySpark a été installé » s’affiche en bas à droite de la fenêtre. Cliquez sur le bouton **Recharger** pour recharger la fenêtre.

   ![PySpark a été installé](./media/hdinsight-for-vscode/pyspark-kernel-installed-successfully.png)

7. Dans la barre de menus, accédez à **Afficher** > **Palette de commandes...** ou utilisez le raccourci **Maj + Ctrl + P**, puis entrez **Python : Sélectionnez Interpréteur pour démarrer le serveur Jupyter**.

   ![sélectionner Interpréteur pour démarrer le serveur Jupyter](./media/hdinsight-for-vscode/select-interpreter-to-start-jupyter-server.png)

8. Sélectionnez l’option Python ci-dessous.

   ![choisit l’option ci-dessous](./media/hdinsight-for-vscode/choose-the-below-option.png)
    
9. Dans la barre de menus, accédez à **Afficher** > **Palette de commandes...** ou utilisez le raccourci **Maj + Ctrl + P**, puis entrez **Développeur : Recharger la fenêtre**.

   ![recharger la fenêtre](./media/hdinsight-for-vscode/reload-window.png)

10. [Connectez-vous](#connect-to-an-azure-account) à votre compte Azure ou liez un cluster si ce n’est déjà fait.

11. Sélectionnez l’ensemble du code, cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez **Spark: PySpark Interactive/Synapse : PySpark Interactive** pour envoyer la requête. 

    ![Menu contextuel pyspark interactive](./media/hdinsight-for-vscode/pyspark-interactive-right-click.png)

12. Si vous n’avez pas spécifié de cluster par défaut, sélectionnez le cluster. Après quelques instants, les résultats **Python Interactive** s’affichent sous un nouvel onglet. Cliquez sur PySpark pour choisir le noyau **PySpark/Synapse Pyspark**. Le code s’exécute sans erreur. Si vous souhaitez basculer vers le noyau Synapse Pyspark, il est recommandé de désactiver la configuration automatique dans le portail Azure. Dans le cas contraire, la relance du cluster et la définition du noyau de Synapse peuvent prendre un certain temps lors de la première utilisation. Les outils vous permettent également d’envoyer un bloc de code au lieu du fichier de script entier à partir du menu contextuel :

    ![Fenêtre Python interactive dans PySpark Interactive](./media/hdinsight-for-vscode/pyspark-interactive-python-interactive-window.png)

13. Entrez **%%info**, puis appuyez sur Maj+Entrée pour afficher les informations sur le travail (facultatif) :

    ![informations travail d’affichage interactif pyspark](./media/hdinsight-for-vscode/pyspark-interactive-view-job-information.png)

L’outil prend également en charge la requête **Spark SQL** :

  ![Affichage des résultats interactifs PySpark](./media/hdinsight-for-vscode/pyspark-ineteractive-select-result.png)


### <a name="perform-interactive-query-in-py-file-using-a--comment"></a>Requête interactive dans le fichier PY à l’aide d’un commentaire #%%

1. Ajoutez **#%%** avant le code Py pour bénéficier de l’expérience de notebook.

   ![Ajout de #%%](./media/hdinsight-for-vscode/run-cell.png)

2. Cliquer sur **Exécuter la cellule**. Au bout de quelques instants, les résultats Python Interactive s’affichent dans un nouvel onglet. Cliquez sur PySpark pour faire basculer le noyau vers PySpark/Synapse PySpark, puis cliquez à nouveau sur **Exécuter la cellule** pour que le code s’exécute correctement.

   ![Résultats Exécuter la cellule](./media/hdinsight-for-vscode/run-cell-get-results.png)

## <a name="leverage-ipynb-support-from-python-extension"></a>Prise en charge IPYNB de l’extension Python

1. Vous pouvez créer un notebook Jupyter par commande à partir de la palette de commandes ou en créant un fichier .ipynb dans votre espace de travail. Pour plus d’informations, consultez [Utilisation de Jupyter Notebook dans Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support).

2. Cliquez sur le bouton **Exécuter la cellule**, suivez les invites pour **Définir le pool Spark par défaut** (il est vivement conseillé de définir un cluster/pool par défaut à chaque ouverture d’un notebook), puis **Rechargez** la fenêtre.

   ![définir le pool Spark par défaut et recharger](./media/hdinsight-for-vscode/set-the-default-spark-pool-and-reload.png)

3. Cliquez sur PySpark pour choisir le noyau **PySpark/Synapse PySpark**, puis cliquez sur **Exécuter la cellule**. Au bout d’un certain temps, le résultat s’affiche.

   ![Résultats Exécuter IPYNB](./media/hdinsight-for-vscode/run-ipynb-file-results.png)


> [!NOTE]
> La version 2020.5.78807 de ms-python et les versions ultérieures ne sont pas prises en charge sur cette extension. Il s’agit d’un [problème connu](#known-issues).

## <a name="submit-pyspark-batch-job"></a>Envoyer le travail de traitement par lots PySpark

1. Rouvrez le dossier **HDexample** évoqué [précédemment](#open-a-work-folder) s’il est fermé.  

2. Créez un fichier **BatchFile.py** en suivant les étapes [précédentes](#open-a-work-folder).

3. Copiez le code suivant et collez-le dans le fichier de script :

   ```python
   from __future__ import print_function
   import sys
   from operator import add
   from pyspark.sql import SparkSession
   if __name__ == "__main__":
       spark = SparkSession\
           .builder\
           .appName("PythonWordCount")\
           .getOrCreate()
   
       lines = spark.read.text('/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv').rdd.map(lambda r: r[0])
       counts = lines.flatMap(lambda x: x.split(' '))\
                  .map(lambda x: (x, 1))\
                   .reduceByKey(add)
       output = counts.collect()
       for (word, count) in output:
           print("%s: %i" % (word, count))
       spark.stop()
   ```

4. [Connectez-vous](#connect-to-an-azure-account) à votre compte Azure ou liez un cluster si ce n’est déjà fait.

5. Cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez **Spark: PySpark Batch** ou **Synapse: PySpark Batch** _.

6. Sélectionnez un cluster ou un pool Spark auquel envoyer le travail PySpark :

   ![Résultat de l’envoi du travail Python](./media/hdinsight-for-vscode/submit-pythonjob-result.png)

Une fois que vous avez envoyé un travail Python, les journaux d'envoi s'affichent dans la fenêtre _ *OUTPUT** (SORTIE) de Visual Studio Code. Les URL des interfaces utilisateur Spark et Yarn s’affichent également. Si vous soumettez le programme de traitement par lots à un pool Apache Spark, l’URL de l’interface utilisateur de l’historique Spark et l’URL de l’interface utilisateur de l’application de travail Spark sont également affichées. Vous pouvez ouvrir l’URL dans un navigateur web pour suivre l’état du travail.

## <a name="integrate-with-hdinsight-identity-broker-hib"></a>Effectuer l’intégration au broker d’ID HDInsight

### <a name="connect-to-your-hdinsight-esp-cluster-with-id-broker-hib"></a>Se connecter à votre cluster HDInsight ESP avec le broker d’ID

Vous pouvez suivre les étapes habituelles pour vous connecter à l’abonnement Azure afin de vous connecter à votre cluster HDInsight ESP avec le broker d’ID. Après vous être connecté, vous verrez la liste des clusters dans Azure Explorer. Pour plus d’instructions, consultez [Se connecter à votre cluster HDInsight](#connect-to-an-azure-account).

### <a name="run-a-hivepyspark-job-on-an-hdinsight-esp-cluster-with-id-broker-hib"></a>Exécution d’un travail Hive/PySpark sur un cluster HDInsight ESP avec broker d’ID (HIB)

Pour exécuter un travail Hive, vous pouvez suivre la procédure habituelle d’envoi d’un travail à un cluster HDInsight ESP avec le broker d’ID (HIB). Pour plus d’instructions, consultez [Envoi de requêtes Hive interactives et de scripts de commandes par lot Hive](#submit-interactive-hive-queries-and-hive-batch-scripts).

Pour exécuter un travail PySpark interactif, vous pouvez suivre la procédure habituelle d’envoi d’un travail à un cluster HDInsight ESP avec le broker d’ID (HIB). Pour plus d’instructions, consultez [Envoi de requêtes PySpark interactives](#submit-interactive-pyspark-queries).

Pour exécuter un programme de traitement par lots PySpark, vous pouvez suivre la procédure habituelle d’envoi d’un travail à un cluster HDInsight ESP avec le broker d’ID (HIB). Pour plus d’instructions, consultez [Envoi d’un programme de traitement par lots PySpark](#submit-pyspark-batch-job).

## <a name="apache-livy-configuration"></a>Configuration d’Apache Livy

La configuration [Apache Livy](https://livy.incubator.apache.org/) est prise en charge. Vous pouvez effectuer la configuration dans le fichier **.VSCode\settings.json** situé dans le dossier de l’espace de travail. Actuellement, la configuration Livy prend uniquement en charge le script Python. Pour plus d’informations, consultez [LISEZ-MOI Livy](https://github.com/cloudera/livy/blob/master/README.rst ).

<a id="triggerlivyconf"></a>**Comment déclencher une configuration Livy**

### <a name="method-1"></a>Méthode 1  

1. Dans la barre de menus, accédez à **Fichier** > **Préférences** > **Paramètres**.
2. Dans la zone **Paramètres de recherche**, entrez **Envoi de travail HDInsight : Livy Conf** (Envoi de travail HDInsight : configuration de Livy).  
3. Sélectionnez **Modifier dans settings.json** pour le résultat de recherche approprié.

### <a name="method-2"></a>Méthode 2

Envoyez un fichier. Vous constatez que le dossier `.vscode` est ajouté automatiquement au dossier de travail. Vous pouvez voir la configuration Livy en sélectionnant **.vscode\settings.json**.

- Paramètres du projet :

  ![Configuration d’Apache Livy HDInsight](./media/hdinsight-for-vscode/hdi-apache-livy-config.png)

  >[!NOTE]
  >Pour les paramètres **driverMemory** et **executorMemory**, définissez la valeur et l’unité. Par exemple : 1 g ou 1 024 m.

- Configurations Livy prises en charge :

  **POST /batches**
  
  **Corps de la demande**

  | name | description | type |
  | --- | --- | --- |
  | fichier | Fichier contenant l’application à exécuter | Chemin (obligatoire) |
  | proxyUser | Utilisateur dont l’identité doit être empruntée lors de l’exécution du travail | String |
  | ClassName | Classe principale Java/Spark de l’application | String |
  | args | Arguments de ligne de commande pour l’application | Liste de chaînes |
  | jars | Fichiers JAR à utiliser dans cette session | Liste de chaînes | 
  | pyFiles | Fichiers Python à utiliser dans cette session | Liste de chaînes |
  | files | Fichiers à utiliser dans cette session | Liste de chaînes |
  | driverMemory | Quantité de mémoire à utiliser pour le processus du pilote | String |
  | driverCores | Nombre de cœurs à utiliser pour le processus du pilote | Int |
  | executorMemory | Quantité de mémoire à utiliser par processus d’exécuteur | String |
  | executorCores | Nombre de cœurs à utiliser pour chaque exécuteur | Int |
  | numExecutors | Nombre d’exécuteurs à lancer pour cette session | Int |
  | archives | Archives à utiliser dans cette session | Liste de chaînes |
  | queue | Nom de la file d’attente YARN vers laquelle effectuer l’envoi| String |
  | name | Nom de cette session | String |
  | conf | Propriétés de configuration de Spark | Mappage clé=valeur |

  **Corps de la réponse** : l’objet Batch créé

  | name | description | type |
  | --- | ---| --- |
  | id | ID de la session | Int |
  | appId | ID d’application de cette session | String |
  | appInfo | Informations détaillées sur l’application | Mappage clé=valeur |
  | log | Lignes du journal | Liste de chaînes |
  | state |État du lot | String |

  > [!NOTE]
  > La configuration Livy attribuée s’affiche dans le volet de sortie quand vous envoyez le script.

## <a name="integrate-with-azure-hdinsight-from-explorer"></a>Intégrer avec Azure HDInsight à partir de l’Explorateur

Vous pouvez voir un aperçu de la table Hive directement dans vos clusters à l’aide de l’explorateur **Azure HDInsight** :

1. [Connectez-vous](#connect-to-an-azure-account) à votre compte Azure, si ce n’est déjà fait.

2. Cliquez sur l’icône **Azure** dans la colonne la plus à gauche.

3. Dans le volet gauche, développez **AZURE : HDINSIGHT**. Les clusters et les abonnements disponibles y sont listés.

4. Développez le cluster pour afficher le schéma de table et la base de données de métadonnées Hive.

5. Cliquez avec le bouton droit sur la table Hive. Par exemple : **hivesampletable**. Sélectionnez **Aperçu**.

   ![Aperçu de la table Hive avec Spark et Hive pour Visual Studio Code](./media/hdinsight-for-vscode/hdinsight-for-vscode-preview-hive-table.png)

6. La fenêtre **Aperçu des résultats** s’ouvre :

   ![Fenêtre Aperçu des résultats avec Spark et Hive pour Visual Studio Code](./media/hdinsight-for-vscode/hdinsight-for-vscode-preview-results-window.png)

- Volet RÉSULTATS

   Vous pouvez enregistrer le résultat complet dans un fichier CSV, JSON ou Excel dans un chemin local ou enregistrer seulement certaines lignes du résultat.

- Volet MESSAGES

  1. Quand le nombre de lignes dans la table est supérieur à 100, le message suivant s’affiche : « Les 100 premières lignes sont affichées dans la table Hive ».
  2. Quand le nombre de lignes dans la table est inférieur ou égal à 100, le message suivant s’affiche : « 60 lignes sont affichées dans la table Hive ».
  3. Quand la table est vide, le message suivant s’affiche : « `0 rows are displayed for Hive table.` »

     >[!NOTE]
     >
     >Sous Linux, installez xclip pour autoriser la copie des données de table.
     >
     >![Spark et Hive pour Visual Studio Code dans Linux](./media/hdinsight-for-vscode/hdinsight-for-vscode-preview-linux-install-xclip.png)

## <a name="additional-features"></a>Fonctionnalités supplémentaires

Les fonctionnalités suivantes sont également prises en charge par Spark et Hive pour Visual Studio Code :

- **Saisie semi-automatique IntelliSense**. Des suggestions de mots clés, de méthodes, de variables et d’autres éléments de programmation s’affichent. Les différentes icônes représentent les différents types d’objets :

    ![Outils Spark et Hive pour Visual Studio Code – Objets IntelliSense](./media/hdinsight-for-vscode/hdinsight-for-vscode-auto-complete-objects.png)

- **Marqueur d’erreur IntelliSense**. Le service de langage souligne les erreurs de saisie dans le script Hive.     
- **Coloration syntaxique**. Le service de langage utilise plusieurs couleurs pour différencier les variables, les mots clés, le type des données, les fonctions et les autres éléments de programmation :

    ![Coloration syntaxique dans l’extension Spark & Hive Tools for Visual Studio Code](./media/hdinsight-for-vscode/hdinsight-for-vscode-syntax-highlights.png)

## <a name="reader-only-role"></a>Rôle Lecteur seul

Les utilisateurs auxquels le rôle de lecteur uniquement a été attribué pour le cluster ne peuvent pas envoyer de travaux au cluster HDInsight, ni visualiser la base de données Hive. Vous devez contacter l’administrateur du cluster afin de mettre à niveau votre rôle pour être [**opérateur de cluster HDInsight**](./hdinsight-migrate-granular-access-cluster-configurations.md#add-the-hdinsight-cluster-operator-role-assignment-to-a-user) dans le [portail Azure](https://portal.azure.com/). Si vous disposez d’informations d’identification Ambari valides, vous pouvez lier manuellement le cluster en suivant les instructions ci-après.

### <a name="browse-the-hdinsight-cluster"></a>Parcourir le cluster HDInsight  

Quand vous sélectionnez l’explorateur Azure HDInsight pour développer un cluster HDInsight, vous êtes invité à lier le cluster si vous disposez du rôle de lecteur uniquement sur le cluster. Utilisez la méthode suivante pour établir une liaison au cluster à l’aide de vos informations d’identification Ambari.

### <a name="submit-the-job-to-the-hdinsight-cluster"></a>Envoyer le travail au cluster HDInsight

Quand vous envoyez un travail à un cluster HDInsight, vous êtes invité à lier le cluster si vous disposez du rôle de lecteur uniquement sur le cluster. Utilisez la procédure suivante pour établir une liaison au cluster à l’aide d’informations d’identification Ambari.

### <a name="link-to-the-cluster"></a>Établir une liaison au cluster

1. Entrez un nom d’utilisateur Ambari valide.
2. Entrez un mot de passe valide.

   ![Nom d’utilisateur pour les outils Spark et Hive pour Visual Studio Code](./media/hdinsight-for-vscode/hdi-azure-hdinsight-azure-username.png)

   ![Outils Spark et Hive pour le mot de passe Visual Studio Code](./media/hdinsight-for-vscode/hdi-azure-hdinsight-azure-password.png)

   > [!NOTE]
   >
   >Vous pouvez utiliser `Spark / Hive: List Cluster` pour vérifier le cluster lié :
   >
   >![Outils Spark et Hive pour lecteur lié Visual Studio Code](./media/hdinsight-for-vscode/list-cluster-result1.png)

## <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

### <a name="browse-a-data-lake-storage-gen2-account"></a>Parcourir un compte Data Lake Storage Gen2

Sélectionnez l’explorateur Azure HDInsight pour développer un compte Data Lake Storage Gen2. Vous êtes invité à entrer la clé d’accès de stockage si votre compte Azure n’a pas accès au stockage Gen2. Une fois la clé d’accès validée, le compte Data Lake Storage Gen2 est développé automatiquement.

### <a name="submit-jobs-to-an-hdinsight-cluster-with-data-lake-storage-gen2"></a>Envoyer des travaux à un cluster HDInsight avec Data Lake Storage Gen2

Envoyez un travail à un cluster HDInsight à l’aide de Data Lake Storage Gen2. Vous êtes invité à entrer la clé d’accès de stockage si votre compte Azure n’a pas accès en écriture au stockage Gen2. Le travail est envoyé avec succès une fois que la clé d’accès est validée.

![Outils Spark et Hive pour clé d’accès Visual Studio Code](./media/hdinsight-for-vscode/hdi-azure-hdinsight-azure-accesskey.png)

> [!NOTE]
>
> Vous pouvez obtenir la clé d’accès du compte de stockage sur le portail Azure. Pour plus d’informations, consultez [Gérer les clés d’accès au compte de stockage](../storage/common/storage-account-keys-manage.md).

## <a name="unlink-cluster"></a>Dissocier le cluster

1. À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes**, puis entrez **Spark / Hive: Unlink a Cluster** (Dissocier un cluster).  

2. Sélectionnez le cluster pour lequel supprimer le lien.  

3. Passez en revue la vue **SORTIE** à des fins de vérification.  

## <a name="sign-out"></a>Se déconnecter  

À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes**, puis entrez **Azure: Se déconnecter**.

## <a name="known-issues"></a>Problèmes connus

### <a name="ms-python-2020578807-version-is-not-supported-on-this-extention"></a>La version 2020.5.78807 de ms-python et les versions ultérieures ne sont pas prises en charge sur cette extension 

« Échec de la connexion au notebook Jupyter » est un problème connu pour la version 2020.5.78807 de Python et les versions ultérieures. Nous recommandons aux utilisateurs d’opter pour la version **[2020.4.76186](https://github.com/microsoft/vscode-python/releases/download/2020.4.76186/ms-python-release.vsix)** de ms-python pour éviter ce problème.

![Problèmes connus](./media/hdinsight-for-vscode/known-issue.png)

## <a name="next-steps"></a>Étapes suivantes

Pour voir une vidéo de démonstration sur l’utilisation de Spark et Hive pour Visual Studio Code, consultez [Spark et Hive pour Visual Studio Code](https://go.microsoft.com/fwlink/?linkid=858706).