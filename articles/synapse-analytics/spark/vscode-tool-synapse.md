---
title: Didacticiel – Spark & Hive Tools pour VSCode (application Spark)
description: Tutoriel - Utiliser Spark & Hive Tools pour VSCode en vue de développer des applications Spark, écrites en Python, et les envoyer à un pool Apache Spark serverless.
services: synapse-analytics
author: hrasheed-msft
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 09/03/2020
ms.openlocfilehash: eb4a4c2c8d1d52690a07b784640d20d96ff2d600
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445684"
---
# <a name="tutorial-create-an-apache-spark-applications-with-vscode-using-a-synapse-workspace"></a>Tutoriel : Créer des applications Apache Spark avec VSCode à l’aide d’un espace de travail Synapse

Découvrez comment utiliser les outils Spark et Hive Apache pour Visual Studio Code. Utilisez les outils pour créer et envoyer des travaux Apache Hive de traitement par lots, des requêtes Hive interactives et des scripts PySpark pour Apache Spark. Tout d’abord, nous décrirons comment installer les outils Spark et Hive dans Visual Studio Code. Ensuite, nous verrons comment envoyer des travaux aux outils Spark et Hive.

Les outils Spark et Hive peuvent être installée sur les plateformes prises en charge par Visual Studio Code. Les différentes plateformes ont les prérequis suivants.

## <a name="prerequisites"></a>Prérequis

Les éléments suivants sont requis pour effectuer les étapes décrites dans cet article :

- Pool Apache Spark serverless. Pour créer un pool Apache Spark serverless, consultez [Créer un pool Apache Spark à l’aide du portail Azure](../../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).
- [Visual Studio Code](https://code.visualstudio.com/).
- [Mono](https://www.mono-project.com/docs/getting-started/install/). Mono est nécessaire uniquement pour les plateformes Linux et MacOS.
- [Un environnement interactif PySpark pour Visual Studio Code](../../hdinsight/set-up-pyspark-interactive-environment.md).
- Un répertoire local. Cet article utilise **C:\HD\Synaseexample**.

## <a name="install-spark--hive-tools"></a>Installer l’extension Spark & Hive Tools

Une fois que vous disposez de tous les éléments prérequis, vous pouvez installer les outils Spark et Hive pour Visual Studio Code en procédant comme suit :

1. Ouvrez Visual Studio Code.

2. Dans la barre de menus, accédez à **Afficher** > **Extensions**.

3. Dans la zone de recherche, entrez **Spark & Hive**.

4. Sélectionnez **Outils Spark et Hive** dans les résultats de la recherche, puis sélectionnez **Installer** :

     ![Installation de Python avec Spark et Hive pour Visual Studio Code](./media/vscode-tool-synapse/install-hdInsight-plugin.png)

5. Sélectionnez **Recharger** si nécessaire.

## <a name="open-a-work-folder"></a>Ouvrir un dossier de travail

Pour ouvrir un dossier de travail et créer un fichier dans Visual Studio Code, procédez comme suit :

1. À partir de la barre de menus, accédez à **Fichier** > **Ouvrir le dossier...**  > **C:\HD\Synaseexample**, puis sélectionnez le bouton **Sélectionner le dossier**. Le dossier s’affiche dans l’affichage **Explorateur** sur la gauche.

2. Dans la vue **Explorateur**, sélectionnez le dossier **Synaseexample**, puis l’icône **Nouveau fichier** à côté du dossier de travail :

     ![icône nouveau fichier visual studio code](./media/vscode-tool-synapse/visual-studio-code-new-file.png)

3. Nommez le nouveau fichier en utilisant l’extension de fichier `.py` (script Spark). Cet exemple utilise **HelloWorld.py**.

## <a name="connect-to-your-spark-pools"></a>Vous connecter à vos pools Spark

Connectez-vous à l’abonnement Azure pour vous connecter à vos pools Spark.

### <a name="sign-in-to-your-azure-subscription"></a>Connectez-vous à votre abonnement Azure :

Suivez ces étapes pour vous connecter à Azure :

1. À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes...** , puis entrez **Azure : Sign In** :

     ![Outils Spark et Hive pour la connexion à Visual Studio Code](./media/vscode-tool-synapse/hdinsight-for-vscode-extension-login.png)

2. Suivez les instructions de connexion pour vous connecter à Azure. Une fois que vous êtes connecté, le nom de votre compte Azure apparaît dans la barre d’état, en bas de la fenêtre Visual Studio Code.

## <a name="set-the-default-spark-pool"></a>Définir le pool Spark par défaut

1. Rouvrez le dossier **Synaseexample** évoqué [précédemment](#open-a-work-folder) s’il est fermé.  

2. Sélectionnez le fichier **HelloWorld.py** créé [précédemment](#open-a-work-folder). Il s’ouvre dans l’éditeur de script.

3. Cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez **Synapse : Définir le pool Spark par défaut**.  

4. [Connectez-vous](#connect-to-your-spark-pools) à votre compte Azure, si ce n’est déjà fait.

5. Sélectionnez un pool Spark comme pool Spark par défaut pour le fichier de script actuel. Les outils mettent automatiquement à jour le fichier de configuration **.VSCode\settings.json** :

     ![Définir une configuration de cluster par défaut](./media/vscode-tool-synapse/set-default-cluster-configuration.png)

## <a name="submit-interactive-synapse-pyspark-queries-to-spark-pool"></a>Envoyer des requêtes Synapse PySpark interactives vers le pool Spark

Les utilisateurs peuvent exécuter Synapse PySpark sur le pool Spark de manière interactive de plusieurs façons :

### <a name="using-the-synapse-pyspark-interactive-command-in-py-file"></a>Commande interactive Synapse PySpark dans le fichier PY
En utilisant la commande interactive PySpark pour envoyer les requêtes, suivez la procédure ci-dessous :

1. Rouvrez le dossier **Synaseexample** évoqué [précédemment](#open-a-work-folder) s’il est fermé.  

2. Créez un fichier **HelloWorld.py** en suivant les étapes [précédentes](#open-a-work-folder).

3. Copiez le code suivant et collez-le dans le fichier de script :

```python
import sys
from operator import add
from pyspark.sql import SparkSession, Row
 
spark = SparkSession\
 .builder\
 .appName("PythonWordCount")\
 .getOrCreate()
 
data = [Row(col1='pyspark and spark', col2=1), Row(col1='pyspark', col2=2), Row(col1='spark vs hadoop', col2=2), Row(col1='spark', col2=2), Row(col1='hadoop', col2=2)]
df = spark.createDataFrame(data)
lines = df.rdd.map(lambda r: r[0])
 
counters = lines.flatMap(lambda x: x.split(' ')) \
 .map(lambda x: (x, 1)) \
 .reduceByKey(add)
 
output = counters.collect()
sortedCollection = sorted(output, key = lambda r: r[1], reverse = True)
 
for (word, count) in sortedCollection:
 print("%s: %i" % (word, count))
```

4. L’invite d’installation du noyau PySpark/Synapse Pyspark s’affiche en bas à droite de la fenêtre. Vous pouvez cliquer sur le bouton **Installer** pour continuer l’installation de PySpark/Synapse Pyspark, ou sur le bouton **Ignorer** pour ignorer cette étape.

     ![Installation du noyau PySpark](./media/vscode-tool-synapse/install-the-pyspark-kernel.png)

5. Si vous avez besoin de l’installer ultérieurement, vous pouvez accéder à **Fichier** > **Préférence** > **Paramètres**, puis décocher **HDInsight : Activer Ignorer l’installation de PySpark** dans les paramètres. 
    
     ![activer ignorer l’installation de PySpark](./media/vscode-tool-synapse/enable-skip-pyspark-installation.png)

6. Si l’installation réussit à l’étape 4, la zone de message « PySpark/Synapse Pyspark a été installé » s’affiche en bas à droite de la fenêtre. Cliquez sur le bouton **Recharger** pour recharger la fenêtre.

     ![PySpark a été installé](./media/vscode-tool-synapse/pyspark-kernel-installed-successfully.png)

7. Dans la barre de menus, accédez à **Afficher** > **Palette de commandes...** ou utilisez le raccourci **Maj + Ctrl + P**, puis entrez **Python : Sélectionnez Interpréteur pour démarrer le serveur Jupyter**.

     ![sélectionner Interpréteur pour démarrer le serveur Jupyter](./media/vscode-tool-synapse/select-interpreter-to-start-jupyter-server.png)

8. Sélectionnez l’option Python ci-dessous.

     ![choisit l’option ci-dessous](./media/vscode-tool-synapse/choose-the-below-option.png)
    
9. Dans la barre de menus, accédez à **Afficher** > **Palette de commandes...** ou utilisez le raccourci **Maj + Ctrl + P**, puis entrez **Développeur : Recharger la fenêtre**.

     ![recharger la fenêtre](./media/vscode-tool-synapse/reload-window.png)

10. [Connectez-vous](#connect-to-your-spark-pools) à votre compte Azure, si ce n’est déjà fait.

11. Sélectionnez l’ensemble du code, cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez **Synapse : PySpark Interactive** pour envoyer la requête. 

     ![Menu contextuel pyspark interactive](./media/vscode-tool-synapse/pyspark-interactive-right-click.png)

12. Si vous n’avez pas spécifié de pool Spark par défaut, sélectionnez le pool Spark. Après quelques instants, les résultats **Python Interactive** s’affichent sous un nouvel onglet. Cliquez sur PySpark pour faire basculer le noyau vers **Synapse PySpark**, puis soumettez à nouveau le code sélectionné pour que le code s’exécute correctement. Les outils vous permettent également d’envoyer un bloc de code au lieu du fichier de script entier à partir du menu contextuel :

     ![interactive](./media/vscode-tool-synapse/pyspark-interactive-python-interactive-window.png)

### <a name="perform-interactive-query-in-py-file-using-a--comment"></a>Requête interactive dans le fichier PY à l’aide d’un commentaire #%%

1. Ajoutez **#%%** avant le code pour bénéficier de l’expérience de notebook.

     ![Ajout de #%%](./media/vscode-tool-synapse/run-cell.png)

2. Cliquer sur **Exécuter la cellule**. Au bout de quelques instants, les résultats Python Interactive s’affichent dans un nouvel onglet. Cliquez sur PySpark pour faire basculer le noyau vers **Synapse PySpark**, puis cliquez à nouveau sur **Exécuter la cellule** pour que le code s’exécute correctement. 

     ![Résultats Exécuter la cellule](./media/vscode-tool-synapse/run-cell-get-results.png)

## <a name="leverage-ipynb-support-from-python-extension"></a>Prise en charge IPYNB de l’extension Python

1. Vous pouvez créer un notebook Jupyter par commande à partir de la palette de commandes ou en créant un fichier .ipynb dans votre espace de travail. Pour plus d’informations, consultez [Utilisation de Jupyter Notebook dans Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support).

2. Cliquez sur le bouton **Exécuter la cellule**, suivez les invites pour **Définir le pool Spark par défaut** (il est vivement conseillé de définir un cluster/pool par défaut à chaque ouverture d’un notebook), puis **Rechargez** la fenêtre.

     ![définir le pool Spark par défaut et recharger](./media/vscode-tool-synapse/set-the-default-spark-pool-and-reload.png)

3. Cliquez sur PySpark pour choisir le noyau **Synapse PySpark**, puis cliquez sur **Exécuter la cellule**. Au bout d’un certain temps, le résultat s’affiche.

     ![Résultats Exécuter IPYNB](./media/vscode-tool-synapse/run-ipynb-file-results.png)


> [!NOTE]
>
>1. La version 2020.5.78807 de ms-python et les versions ultérieures ne sont pas prises en charge sur cette extension. Il s’agit d’un [problème connu](#known-issues).
>  
>2. Basculer vers le noyau Synapse Pyspark. La désactivation de la configuration automatique dans le portail Azure est encouragée. Dans le cas contraire, la relance du cluster et la définition du noyau de Synapse peuvent prendre un certain temps lors de la première utilisation. 
>
>    ![paramètres automatiques](./media/vscode-tool-synapse/auto-settings.png)

## <a name="submit-pyspark-batch-job-to-spark-pool"></a>Envoyer le travail de traitement par lots PySpark vers le pool Spark

1. Rouvrez le dossier **Synaseexample** évoqué [précédemment](#open-a-work-folder) s’il est fermé.  

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

4. [Connectez-vous](#connect-to-your-spark-pools) à votre compte Azure, si ce n’est déjà fait.

5. Cliquez avec le bouton droit sur l’éditeur de script, puis sélectionnez **Synapse : PySpark Batch**.

6. Sélectionnez un pool Spark auquel envoyer le travail PySpark :

     ![Résultat de l’envoi du travail Python](./media/vscode-tool-synapse/submit-pythonjob-result.png)

Une fois que vous avez envoyé un programme de traitement par lots, les journaux d’envoi s’affichent dans la fenêtre **OUTPUT** (SORTIE) de Visual Studio Code. L’URL **Spark UI** et l’URL **Spark Job Application UI** s’affichent également. Vous pouvez ouvrir l’URL dans un navigateur web pour suivre l’état du travail.

## <a name="access-and-manage-synapse-workspace"></a>Accéder à l’espace de travail Synapse et le gérer

Vous pouvez effectuer différentes opérations dans Azure Explorer au sein des outils Spark et Hive pour VSCode. À partir de l’explorateur Azure.

![image azure](./media/vscode-tool-synapse/azure.png)

### <a name="launch-workspace"></a>Lancer l’espace de travail

1. Dans l’explorateur Azure, accédez à **SYNAPSE**, développez et affichez la liste des abonnements synapse.

     ![explorateur synapse](./media/vscode-tool-synapse/synapse-explorer.png)

2. Cliquez sur Abonnement dans l’espace de travail Synapse, développez-le et affichez la liste des espaces de travail.

3. Cliquez avec le bouton droit sur un espace de travail, puis sélectionnez **Afficher les applications Apache Spark**, la page de l’application Apache Spark du site Web de Synapse Studio s’ouvre.

     ![cliquez avec le bouton droit sur l’espace de travail](./media/vscode-tool-synapse/right-click-on-workspace.png)

     ![application synapse studio](./media/vscode-tool-synapse/synapse-studio-application.png)

4. Développez un espace de travail, **Stockage par défaut** et **Pools Spark** s’affichent.

5. Cliquez avec le bouton droit sur **Stockage par défaut**. **Copier le chemin complet** et **Ouvrir dans Synapse Studio** s’affichent. 

     ![cliquez avec le bouton droit sur stockage par défaut](./media/vscode-tool-synapse/right-click-on-default-storage.png)

     - Cliquez sur **Copier le chemin complet**, l’URL du compte ADLS Gen2 principale sera copié, vous pouvez la copier où vous le souhaitez.

     - Cliquez sur **Ouvrir dans Synapse Studio**, le compte de stockage principal sera ouvert dans Synapse Studio.

     ![stockage par défaut dans synapse studio](./media/vscode-tool-synapse/default-storage-in-synapse-studio.png)

6. Développez le **Stockage par défaut**, le compte de stockage principal s’affiche.

7. Développez les **Pools Spark**, tous les pools Spark dans l’espace de travail s’affichent.

     ![développez le pool de stockage](./media/vscode-tool-synapse/expand-storage-pool.png)


## <a name="known-issues"></a>Problèmes connus

### <a name="ms-python-2020578807-version-is-not-supported-on-this-extention"></a>La version 2020.5.78807 de ms-python et les versions ultérieures ne sont pas prises en charge sur cette extension 

« Échec de la connexion au notebook Jupyter » est un problème connu pour la version 2020.5.78807 de Python et les versions ultérieures. Nous recommandons aux utilisateurs d’opter pour la version **[2020.4.76186](https://github.com/microsoft/vscode-python/releases/download/2020.4.76186/ms-python-release.vsix)** de ms-python pour éviter ce problème.

![Problèmes connus](./media/vscode-tool-synapse/known-issue.png)


## <a name="next-steps"></a>Étapes suivantes

- [Azure Synapse Analytics](../overview-what-is.md)
- [Créer un pool Apache Spark pour un espace de travail Azure Synapse Analytics](../../synapse-analytics/quickstart-create-apache-spark-pool-studio.md)
