---
title: Gérer les dépendances d’application Spark sur Azure HDInsight
description: Cet article présente la gestion des dépendances Spark dans le cluster HDInsight Spark pour les applications PySpark et Scala.
author: yanancai
ms.author: yanacai
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 09/09/2020
ms.openlocfilehash: b2cd50b1b35b87b1a11301ddc36ac355bef20dc4
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780613"
---
# <a name="manage-spark-application-dependencies"></a>Gérer les dépendances d’application Spark

Dans cet article, vous allez apprendre à gérer les dépendances de vos applications Spark s’exécutant sur HDInsight. Nous abordons les types Scala et PySpark pour l’application Spark et l’étendue du cluster.

Utilisez les liens rapides pour accéder à la section en fonction de votre cas d’usage :
* [Configurer les dépendances jar du travail Spark à l’aide de Jupyter Notebook](#use-jupyter-notebook)
* [Configurer les dépendances jar du travail Spark à l’aide d’Azure Toolkit for IntelliJ](#use-azure-toolkit-for-intellij)
* [Configurer les dépendances jar pour le cluster Spark](#jar-libs-for-cluster)
* [Gérer les dépendances jar de façon sécurisée](#safely-manage-jar-dependencies)
* [Configurer des packages Python de travail Spark à l’aide de Jupyter Notebook](#use-jupyter-notebook-1)
* [Gérer en toute sécurité les packages Python pour le cluster Spark](#python-packages-for-cluster)

## <a name="jar-libs-for-one-spark-job"></a>Bibliothèques jar pour un travail Spark
### <a name="use-jupyter-notebook"></a>Utiliser Jupyter Notebook
Lorsqu’une session Spark démarre dans Jupyter Notebook sur le noyau Spark pour Scala, vous pouvez configurer des packages à partir de ce qui suit :

* [Référentiel Maven](https://search.maven.org/) ou packages proposés par la communauté pour [Packages Spark](https://spark-packages.org/).
* Fichiers jar stockés sur le stockage principal de votre cluster.

Vous allez utiliser la commande magique `%%configure` pour configurer le bloc-notes afin d’utiliser un package externe. Dans les blocs-notes utilisant des packages externes, veillez à appeler la commande magique `%%configure` dans la première cellule de code. Cela garantit que le noyau est configuré pour utiliser le package avant le démarrage de la session.

>
>[!IMPORTANT]  
>Si vous oubliez de configurer le noyau dans la première cellule, vous pouvez utiliser `%%configure` avec le paramètre `-f`. Toutefois, cette opération redémarrera la session et entraînera la perte de toute la progression.

**Exemple de packages issus du référentiel Maven ou des packages Spark**

Après avoir localisé le package du référentiel Maven, rassemblez les valeurs pour **GroupId**, **ArtifactId** et **Version**. Concaténez les trois valeurs séparées par deux-points ( **:** ).

   ![Concaténer le schéma du package](./media/apache-spark-manage-dependencies/spark-package-schema.png "Concaténer le schéma du package")

Vérifiez que les valeurs que vous collectez correspondent à votre cluster. Dans ce cas, nous utilisons le package du connecteur Spark Cosmos DB pour Scala 2.11 et Spark 2.3 pour le cluster Spark HDInsight 3.6. En cas de doute, exécutez `scala.util.Properties.versionString` dans la cellule de code du noyau Spark pour accéder à la version Scala du cluster. Exécutez `sc.version` pour accéder à la version Spark du cluster.

```
%%configure { "conf": {"spark.jars.packages": "com.microsoft.azure:azure-cosmosdb-spark_2.3.0_2.11:1.3.3" }}
```

**Exemple de fichiers jar stockés sur le stockage principal**

Utilisez le [schéma d’URI](../hdinsight-hadoop-linux-information.md#URI-and-scheme) pour les fichiers jar de votre principal espace de stockage de clusters. Il s’agirait de `wasb://` pour Stockage Azure, de `abfs://` pour Azure Data Lake Storage Gen2 ou de `adl://` pour Azure Data Lake Storage Gen1. Si l’option de transfert sécurisé était activée pour Stockage Azure ou Data Lake Storage Gen2, l’URI serait `wasbs://` ou `abfss://`. Consultez [Transfert sécurisé](../../storage/common/storage-require-secure-transfer.md).

Utilisez une liste séparée par des virgules de chemins jar pour plusieurs fichiers jar ; les modèles Glob sont autorisés. Les fichiers jar sont inclus dans les classpaths du pilote et de l’exécuteur.

```
%%configure { "conf": {"spark.jars": "wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/azure-cosmosdb-spark_2.3.0_2.11-1.3.3.jar" }}
```

Après avoir configuré les packages externes, vous pouvez exécuter l’importation dans la cellule de code pour vérifier si les packages ont été correctement placés.

```scala
import com.microsoft.azure.cosmosdb.spark._
```

### <a name="use-azure-toolkit-for-intellij"></a>Utiliser Azure Toolkit for IntelliJ
Le [plug-in Azure Toolkit for IntelliJ](./apache-spark-intellij-tool-plugin.md) offre une expérience d’interface utilisateur pour envoyer une application Spark Scala à un cluster HDInsight. Il fournit des propriétés `Referenced Jars` et `Referenced Files` pour configurer les chemins d’accès aux bibliothèques jar lors de l’envoi de l’application Spark. Consultez plus d’informations sur [l’utilisation du plug-in Azure Toolkit for IntelliJ pour HDInsight](./apache-spark-intellij-tool-plugin.md#run-a-spark-scala-application-on-an-hdinsight-spark-cluster).

![Boîte de dialogue Spark Submission (Envoi Spark)](./media/apache-spark-intellij-tool-plugin/hdi-submit-spark-app-02.png)

## <a name="jar-libs-for-cluster"></a>Bibliothèques jar pour le cluster
Dans certains cas, vous souhaiterez peut-être configurer les dépendances jar au niveau du cluster pour permettre la configuration de chaque application avec les mêmes dépendances par défaut. Cette approche consiste à ajouter vos chemins jar au pilote Spark et au chemin de classe de l’exécuteur.

1. Exécutez les exemples d’actions de script ci-dessous pour copier les fichiers jar entre le stockage principal `wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/*` et le système de fichiers local du cluster `/usr/libs/sparklibs`. Cette étape est nécessaire car Linux utilise `:` pour séparer la liste de chemins de classe, mais HDInsight prend uniquement en charge les chemins de stockage présentant un schéma de type `wasb://`. Le chemin de stockage distant ne fonctionnera pas correctement si vous l’ajoutez directement au chemin de classe.

    ```bash
    sudo mkdir -p /usr/libs/sparklibs
    sudo hadoop fs -copyToLocal wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/*.* /usr/libs/sparklibs
    ```

2. Modifiez la configuration du service Spark depuis Ambari pour mettre à jour le chemin de classe. Accédez à **Ambari > Spark > Configurations > Custom Spark2-defaults (Valeurs Spark2-defaults personnalisées)** . **Ajoutez la propriété** comme suit. Utilisez `:` pour séparer les chemins s’il vous faut en ajouter plusieurs. Les modèles Glob sont autorisés.

    ```
    spark.driver.extraClassPath=/usr/libs/sparklibs/*
    spark.executor.extraClassPath=/usr/libs/sparklibs/*
    ```

   ![Modifier la configuration Spark par défaut](./media/apache-spark-manage-dependencies/change-spark-default-config.png "Modifier la configuration Spark par défaut")

3. Enregistrez les configurations modifiées et redémarrez les services impactés.

   ![Redémarrer les services impactés](./media/apache-spark-manage-dependencies/restart-impacted-services.png "Redémarrer les services impactés")

Vous pouvez automatiser la procédure à l’aide d'[actions de script](../hdinsight-hadoop-customize-cluster-linux.md). L’action de script permettant d’[ajouter des bibliothèques Hive personnalisées](https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh) constitue une bonne référence. Lorsque vous modifiez les configurations du service Spark, veillez à utiliser les API Ambari plutôt que de modifier directement les fichiers de configuration. 

## <a name="safely-manage-jar-dependencies"></a>Gérer les dépendances jar de façon sécurisée
Le cluster HDInsight est doté de dépendances jar intégrées, et les mises à jour de ces versions jar interviennent de temps à autre. Pour éviter les conflits de version entre les fichiers jar intégrés et les fichiers jar que vous utilisez à des fins de référence, envisagez [l’ombrage de vos dépendances d’application](./safely-manage-jar-dependency.md).

## <a name="python-packages-for-one-spark-job"></a>Packages Python pour un travail Spark
### <a name="use-jupyter-notebook"></a>Utiliser Jupyter Notebook
Le noyau PySpark Jupyter Notebook HDInsight ne prend pas en charge l’installation directe de packages Python à partir du référentiel de packages PyPi ou Anaconda. Si vous disposez de dépendances de `.zip`, `.egg`ou `.py` , et que vous souhaitez les référencer pour une session Spark, suivez la procédure ci-dessous :

1. Exécutez les exemples d’actions de script ci-dessous pour copier les fichiers `.zip`, `.egg` ou `.py` du stockage principal `wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/*` vers le système de fichiers local du cluster `/usr/libs/pylibs`. Cette étape est nécessaire car Linux utilise `:` pour séparer la liste de chemins de recherche, mais HDInsight prend uniquement en charge les chemins de stockage présentant un schéma de type `wasb://`. Le chemin d’accès de stockage distant ne fonctionnera pas correctement lorsque vous utiliserez `sys.path.insert`.

    ```bash
    sudo mkdir -p /usr/libs/pylibs
    sudo hadoop fs -copyToLocal wasb://mycontainer@mystorageaccount.blob.core.windows.net/libs/*.* /usr/libs/pylibs
    ```

2. Dans votre notebook, exécutez le code ci-dessous dans une cellule de code avec le noyau PySpark :

   ```python
   import sys
   sys.path.insert(0, "/usr/libs/pylibs/pypackage.zip")
   ```

3. Exécutez `import` pour vous assurer que vos packages ont bien été inclus.  

## <a name="python-packages-for-cluster"></a>Packages Python pour le cluster
Vous pouvez installer des packages Python entre Anaconda et le cluster à l’aide de la commande conda via des actions de script. Les packages installés se situent au niveau du cluster et portent sur toutes les applications. 

Le cluster HDInsight Spark dispose de deux installations Python intégrées, Anaconda Python 2.7 et Python 3.5. Pour en savoir plus sur les paramètres Python par défaut et sur la façon d’installer en toute sécurité des packages Python externes sans rompre le cluster, consultez [Gérer en toute sécurité les dépendances Python pour votre cluster](./apache-spark-python-package-installation.md).
