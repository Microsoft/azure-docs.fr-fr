---
title: Environnement interactif PySpark avec Azure HDInsight Tools
description: Découvrez comment utiliser Azure HDInsight Tools pour Visual Studio Code pour créer et envoyer des requêtes et des scripts.
keywords: VScode,Azure HDInsight Tools,Hive,Python,PySpark,Spark,HDInsight,Hadoop,LLAP,Interactive Hive,Interactive Query
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020, devx-track-python
ms.date: 04/23/2020
ms.openlocfilehash: 7cf86824bef5b6f521df0f0446d5337e2b7c418a
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130134"
---
# <a name="set-up-the-pyspark-interactive-environment-for-visual-studio-code"></a>Définir l’environnement interactif de PySpark pour Visual Studio Code

Les étapes suivantes expliquent comment configurer l'environnement interactif PySpark dans VS Code. Cette étape concerne uniquement les utilisateurs autres que Windows.

Nous utilisons la commande **python/pip** pour créer un environnement virtuel au niveau de votre chemin d'accueil. Si vous souhaitez utiliser une autre version, vous devez modifier manuellement la version par défaut de la commande **python/pip**. Pour plus de détails, voir [update-alternatives](https://linux.die.net/man/8/update-alternatives).

1. Installez [Python](https://www.python.org/downloads/) et [pip](https://pip.pypa.io/en/stable/installing/).

   * Installez Python à partir de [https://www.python.org/downloads/](https://www.python.org/downloads/). 
   * Installez pip depuis [https://pip.pypa.io/en/stable/installing](https://pip.pypa.io/en/stable/installing/) (si non installé à partir de l'installation de Python).
   * Si vous le souhaitez, utilisez les commandes `python --version` et `pip --version` pour vérifier que Python et pip, respectivement, sont bien installés. 

     > [!NOTE]
     > Il est recommandé d'installer manuellement Python plutôt que d'utiliser la version par défaut de macOS.

2. Installez **virtualenv** en exécutant la commande ci-dessous.

   ```bash
   pip install virtualenv
   ```

## <a name="other-packages"></a>Autres packages

Dans Linux, si vous rencontrez le message d’erreur ci-dessous, installez les packages nécessaires en exécutant les deux commandes suivantes.

   ![Installer le package libkrb5 pour Python](./media/set-up-pyspark-interactive-environment/install-libkrb5-package.png)

```bash
sudo apt-get install libkrb5-dev
```

```bash
sudo apt-get install python-dev
```

Redémarrez VSCode, revenez à l’éditeur VSCode, puis exécutez la commande **Spark : PySpark Interactive**.

## <a name="next-steps"></a>Étapes suivantes

### <a name="demo"></a>Démonstration

* HDInsight pour VS Code : [Vidéo](https://go.microsoft.com/fwlink/?linkid=858706)

### <a name="tools-and-extensions"></a>Outils et extensions

* [Utiliser Azure HDInsight Tools pour Visual Studio Code](hdinsight-for-vscode.md)
* [Utiliser Azure Toolkit for IntelliJ pour créer et soumettre des applications Apache Spark Scala](spark/apache-spark-intellij-tool-plugin.md)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster (Installer Jupyter sur un ordinateur et se connecter au cluster Spark sur HDInsight)](spark/apache-spark-jupyter-notebook-install-locally.md)
