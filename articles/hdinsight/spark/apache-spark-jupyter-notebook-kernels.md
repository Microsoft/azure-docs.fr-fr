---
title: Noyaux pour bloc-notes Jupyter sur les clusters Spark dans Azure HDInsight
description: Découvrez les noyaux PySpark, PySpark3 et Spark pour bloc-notes Jupyter qui sont disponibles avec les clusters Spark sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 04/24/2020
ms.openlocfilehash: 8c3993d8208a9a9e2ab54be44d88de0b20a2e586
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86084713"
---
# <a name="kernels-for-jupyter-notebook-on-apache-spark-clusters-in-azure-hdinsight"></a>Noyaux pour Jupyter Notebook sur des clusters Apache Spark dans Azure HDInsight

Les clusters Spark HDInsight fournissent des noyaux utilisables avec Jupyter Notebook sur [Apache Spark](./apache-spark-overview.md) pour tester des applications. Un noyau est un programme qui exécute et interprète votre code. Les trois noyaux sont les suivants :

- **PySpark**, pour les applications écrites en Python2.
- **PySpark3**, pour les applications écrites en Python3.
- **Spark**, pour les applications écrites en Scala.

Dans cet article, vous allez apprendre à utiliser ces noyaux et découvrir les avantages de leur utilisation.

## <a name="prerequisites"></a>Prérequis

Un cluster Apache Spark dans HDInsight. Pour obtenir des instructions, consultez [Création de clusters Apache Spark dans Azure HDInsight](apache-spark-jupyter-spark-sql.md).

## <a name="create-a-jupyter-notebook-on-spark-hdinsight"></a>Créer un bloc-notes Jupyter sur Spark HDInsight

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre projet Spark.  Pour obtenir des instructions, consultez la page [Énumération et affichage des clusters](../hdinsight-administer-use-portal-linux.md#showClusters). La page **Vue d’ensemble** s’ouvre.

2. Dans la zone **Tableaux de bord du cluster** de la page **Vue d’ensemble**, sélectionnez **Jupyter Notebook**. Si vous y êtes invité, entrez les informations d’identification d’administrateur pour le cluster.

    ![Notebook Jupyter sur Apache Spark](./media/apache-spark-jupyter-notebook-kernels/hdinsight-spark-open-jupyter-interactive-spark-sql-query.png "Notebook Jupyter sur Spark")
  
   > [!NOTE]  
   > Vous pouvez également atteindre le bloc-notes Jupyter sur le cluster Spark en ouvrant l’URL suivante dans votre navigateur. Remplacez **CLUSTERNAME** par le nom de votre cluster.
   >
   > `https://CLUSTERNAME.azurehdinsight.net/jupyter`

3. Sélectionnez **Nouveau**, puis **Pyspark**, **PySpark3** ou **Spark** pour créer un notebook. Utilisez le noyau Spark pour les applications Scala, le noyau PySpark pour les applications Python2 et le noyau PySpark3 pour les applications Python3.

    ![Noyaux pour le Notebook Jupyter sur Spark](./media/apache-spark-jupyter-notebook-kernels/kernel-jupyter-notebook-on-spark.png "Noyaux pour le Notebook Jupyter sur Spark")

4. Un bloc-notes s’ouvre avec le noyau que vous avez sélectionné.

## <a name="benefits-of-using-the-kernels"></a>Avantages de l’utilisation des noyaux

Voici quelques avantages liés à l’utilisation des nouveaux noyaux avec bloc-notes Jupyter sur des clusters Spark HDInsight.

- **Contextes prédéfinis**. Avec les noyaux **PySpark**, **PySpark3** ou **Spark**, vous n’avez pas besoin de définir les contextes Spark ou Hive explicitement avant de commencer à utiliser vos applications. Ces contextes sont disponibles par défaut. Ces contextes sont les suivants :

  - **sc** : pour le contexte Spark
  - **sqlContext** : pour le contexte Hive

    Par conséquent, vous **n’avez pas** à exécuter d’instructions telles que les suivantes pour définir les contextes :

    ```sql
    sc = SparkContext('yarn-client')
    sqlContext = HiveContext(sc)
    ```

    En revanche, vous pouvez utiliser directement les contextes prédéfinis dans votre application.

- **Commandes magiques de cellule**. Le noyau PySpark fournit certaines « commandes magiques » prédéfinies, qui sont des commandes spéciales que vous pouvez appeler avec `%%` (par exemple, `%%MAGIC` `<args>`). La commande magique doit se trouver au tout début d’une cellule de code et autoriser plusieurs lignes de contenu. Le mot magic doit être le premier mot de la cellule. Ajouter quoi que ce soit avant la commande magique, même des commentaires, provoque une erreur.     Pour plus d’informations sur les commandes magiques, cliquez [ici](https://ipython.readthedocs.org/en/stable/interactive/magics.html).

    Le tableau suivant répertorie les différentes commandes magiques disponibles par le biais des noyaux.

   | Commande magique | Exemple | Description |
   | --- | --- | --- |
   | help |`%%help` |Génère une table de toutes les commandes magiques disponibles, accompagnées d’un exemple et d’une description |
   | info |`%%info` |Génère des informations de session pour le point de terminaison Livy actuel |
   | CONFIGURER |`%%configure -f`<br>`{"executorMemory": "1000M"`<br>`"executorCores": 4`} |Configure les paramètres de création d’une session. L’indicateur de forçage (`-f`) est obligatoire si une session a déjà été créée, afin de garantir que la session est supprimée et recréée. Consultez la section [POST /sessions Request Body de Livy](https://github.com/cloudera/livy#request-body) pour obtenir la liste des paramètres valides. Les paramètres doivent être passés en tant que chaîne JSON et être spécifiés sur la ligne suivant la commande magique, comme indiqué dans l’exemple de colonne. |
   | sql |`%%sql -o <variable name>`<br> `SHOW TABLES` |Exécute une requête Hive sur sqlContext. Si le paramètre `-o` est passé, le résultat de la requête est conservé dans le contexte Python %%local en tant que trame de données [Pandas](https://pandas.pydata.org/) . |
   | local |`%%local`<br>`a=1` |Tout le code dans les lignes suivantes est exécuté localement. Le code doit être un code Python2 valide, peu importe quel noyau vous utilisez. Donc, même si vous avez sélectionné les noyaux **PySpark3** ou **Spark** en créant le notebook, si vous utilisez la commande magique `%%local` dans une cellule, cette cellule doit uniquement comporter un code Python2 valide. |
   | logs |`%%logs` |Génère les journaux d’activité de la session Livy en cours. |
   | supprimer |`%%delete -f -s <session number>` |Supprime une session spécifique du point de terminaison Livy actuel. Vous ne pouvez pas supprimer la session qui est démarrée pour le noyau lui-même. |
   | cleanup |`%%cleanup -f` |Supprime toutes les sessions pour le point de terminaison Livy actuel, y compris la session de ce bloc-notes. L’indicateur de forçage -f est obligatoire. |

   > [!NOTE]  
   > Outre les commandes magiques ajoutées par le noyau PySpark, vous pouvez également utiliser les [commandes magiques IPython intégrées](https://ipython.org/ipython-doc/3/interactive/magics.html#cell-magics), notamment `%%sh`. Vous pouvez utiliser la commande magique `%%sh` pour exécuter des scripts et des blocs de code sur le nœud principal du cluster.

- **Visualisation automatique**. Le noyau Pyspark visualise automatiquement la sortie des requêtes Hive et SQL. Vous pouvez choisir entre plusieurs types de visualisations, notamment tableau, secteurs, courbes, aires et barres.

## <a name="parameters-supported-with-the-sql-magic"></a>Paramètres pris en charge avec la commande magique %%sql

La commande magique `%%sql` prend en charge différents paramètres qui vous permettent de contrôler le type de sortie que vous recevez quand vous exécutez des requêtes. Le tableau suivant répertorie les paramètres de sortie.

| Paramètre | Exemple | Description |
| --- | --- | --- |
| -o |`-o <VARIABLE NAME>` |Utilisez ce paramètre pour conserver le résultat de la requête dans le contexte Python %%local en tant que trame de données [Pandas](https://pandas.pydata.org/) . Le nom de la variable dataframe est le nom de variable que vous spécifiez. |
| -q |`-q` |Utilisez ce paramètre pour désactiver les visualisations pour la cellule. Si vous ne voulez pas visualiser automatiquement le contenu d’une cellule et préférez simplement capturer le contenu comme une trame de données, utilisez `-q -o <VARIABLE>`. Si vous souhaitez désactiver les visualisations sans capturer les résultats (par exemple, pour exécuter une requête SQL, comme une instruction `CREATE TABLE`), utilisez `-q` sans spécifier d’argument `-o`. |
| -M |`-m <METHOD>` |**METHOD** prend la valeur **take** ou **sample** (**take** est la valeur par défaut). Si la méthode est **`take`** , le noyau sélectionne des éléments à partir du haut du jeu de données de résultats spécifié par la valeur MAXROWS (décrite plus bas dans ce tableau). Si la méthode est **sample**, le noyau échantillonne de façon aléatoire les éléments du jeu de données en fonction du paramètre `-r` (décrit ci-après dans ce tableau). |
| -r |`-r <FRACTION>` |Ici, **FRACTION** est un nombre à virgule flottante compris entre 0,0 et 1,0. Si l’exemple de méthode de la requête SQL est `sample`, le noyau échantillonne de façon aléatoire la fraction spécifiée des éléments du jeu de résultats. Par exemple, si vous exécutez une requête SQL avec les arguments `-m sample -r 0.01`, 1 % des lignes de résultat sont échantillonnées de façon aléatoire. |
| -n |`-n <MAXROWS>` |**MAXROWS** est une valeur entière. Le noyau limite le nombre de lignes de la sortie au nombre défini par **MAXROWS**. Si **MAXROWS** est un nombre négatif comme **-1**, le nombre de lignes dans le jeu de résultats n’est pas limité. |

**Exemple :**

```sql
%%sql -q -m sample -r 0.1 -n 500 -o query2
SELECT * FROM hivesampletable
```

L’instruction ci-dessus effectue les actions suivantes :

- Elle sélectionne tous les enregistrements présents dans **hivesampletable**.
- Comme nous utilisons le paramètre -q, elle désactive la visualisation automatique.
- Comme nous utilisons `-m sample -r 0.1 -n 500` , elle échantillonne de façon aléatoire 10 % des lignes présentes dans hivesampletable et limite la taille du jeu de résultats à 500 lignes.
- Enfin, comme nous avons utilisé `-o query2` , elle enregistre également la sortie dans une trame de données appelée **query2**.

## <a name="considerations-while-using-the-new-kernels"></a>Points à prendre en compte lors de l'utilisation des nouveaux noyaux

Quel que soit le noyau que vous utilisez, laisser les blocs-notes s’exécuter consomme vos ressources de cluster.  Étant donné que les contextes sont prédéfinis, le simple fait de quitter les notebooks avec ces noyaux n’arrête pas le contexte. Par conséquent, les ressources du cluster restent en cours d’utilisation. Une bonne pratique consiste à utiliser l’option **Fermer et arrêter** à partir du menu **Fichier** du notebook lorsque vous avez fini d’utiliser le notebook. Cette fermeture supprime le contexte puis ferme le bloc-notes.

## <a name="where-are-the-notebooks-stored"></a>Où sont stockés les blocs-notes ?

Si votre cluster utilise le stockage Azure comme compte de stockage par défaut, les bloc-notes Jupyter sont enregistrés dans le compte de stockage sous le dossier **/HdiNotebooks**.  Les blocs-notes, les fichiers texte et les dossiers que vous créez dans Jupyter sont accessibles à partir du compte de stockage.  Par exemple, si vous utilisez Jupyter pour créer un dossier **`myfolder`** et un bloc-notes **myfolder/mynotebook.ipynb**, vous pouvez accéder à ce bloc-notes dans `/HdiNotebooks/myfolder/mynotebook.ipynb` au sein du compte de stockage.  L’inverse est également vrai : si vous chargez un bloc-notes directement dans votre compte de stockage dans `/HdiNotebooks/mynotebook1.ipynb`, le bloc-notes est également accessible à partir de Jupyter.  Les blocs-notes sont conservés dans le compte de stockage même après la suppression du cluster.

> [!NOTE]  
> Les clusters HDInsight utilisant Azure Data Lake Storage comme stockage par défaut ne stockent pas les bloc-notes dans le stockage associé.

Les blocs-notes sont enregistrés dans le compte de stockage dans un mode compatible avec [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html). Si vous utilisez SSH dans le cluster, vous pouvez exécuter des commandes de gestion des fichiers :

| Commande | Description |
|---------|-------------|
| `hdfs dfs -ls /HdiNotebooks` | # Répertorier tous les éléments du répertoire racine : tout ce qui se trouve dans ce répertoire est visible pour Jupyter à partir de la page d’accueil |
| `hdfs dfs –copyToLocal /HdiNotebooks` | # Télécharger le contenu du dossier HdiNotebooks|
| `hdfs dfs –copyFromLocal example.ipynb /HdiNotebooks` | # Télécharger un bloc-notes exemple. ipynb dans le dossier racine afin qu’il soit visible à partir de Jupyter |

Que le cluster utilise le stockage Azure ou Azure Data Lake Storage comme compte de stockage par défaut, les blocs-notes sont aussi enregistrés sur le nœud principal du cluster à l’emplacement `/var/lib/jupyter`.

## <a name="supported-browser"></a>Navigateur pris en charge

Les blocs-notes Jupyter sur clusters Spark HDInsight sont pris en charge uniquement sur Google Chrome.

## <a name="suggestions"></a>Suggestions

Les nouveaux noyaux sont en phase d’évolution et gagneront en maturité avec le temps. Les API pourront donc évoluer au fur et à mesure des évolutions des noyaux. Nous aimerions recevoir vos commentaires concernant l'utilisation de ces nouveaux noyaux. Ce retour nous est utile pour préparer la version finale de ces noyaux. Vous pouvez laisser vos commentaires dans la section **Commentaires** en bas de cet article.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble : Apache Spark sur Azure HDInsight](apache-spark-overview.md)
- [Utiliser des blocs-notes Apache Zeppelin avec un cluster Apache Spark sur HDInsight](apache-spark-zeppelin-notebook.md)
- [Utiliser des packages externes avec les blocs-notes Jupyter](apache-spark-jupyter-notebook-use-external-packages.md)
- [Install Jupyter on your computer and connect to an HDInsight Spark cluster (Installer Jupyter sur un ordinateur et se connecter au cluster Spark sur HDInsight)](apache-spark-jupyter-notebook-install-locally.md)
