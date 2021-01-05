---
title: Utiliser Apache Pig
titleSuffix: Azure HDInsight
description: Découvrez comment utiliser Pig avec Apache Hadoop sur HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 01/28/2020
ms.openlocfilehash: 9474a1845898e3fda0c810314f41f6f4deabc96b
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92538729"
---
# <a name="use-apache-pig-with-apache-hadoop-on-hdinsight"></a>Utiliser Apache Pig avec Apache Hadoop sur HDInsight

Découvrez comment utiliser [Apache Pig](https://pig.apache.org/) avec HDInsight.

Apache Pig est une plateforme qui permet de créer des programmes pour Apache Hadoop dans un langage procédural appelé *Pig Latin* . Pig est une alternative à Java, dédiée à la création de solutions *MapReduce* , qui est incluse avec Azure HDInsight. Utilisez le tableau suivant pour découvrir les différentes façons dont Pig peut être utilisé avec HDInsight :

## <a name="why-use-apache-pig"></a><a id="why"></a>Pourquoi utiliser Apache Pig

Lun des défis de traitement de données à l'aide de MapReduce dans Hadoop consiste à implémenter votre logique de traitement en utilisant uniquement un mappage et une fonction de réduction. Pour un traitement complexe, vous devez souvent interrompre le traitement lors de plusieurs opérations MapReduce qui sont chaînées ensemble pour produire le résultat souhaité.

Pig permet de définir un traitement comme une série de transformations via lesquelles les données circulent pour produire le résultat souhaité.

Le langage Pig Latin vous permet de décrire le flux de données provenant d’une entrée brute, via une ou plusieurs transformations, pour produire le résultat souhaité. Les programmes Pig Latin suivent le modèle général suivant :

* **Chargement** : lecture des données à manipuler dans le système de fichiers.

* **Transformation** : manipulation des données.

* **Sortie ou stockage** : affichage du résultat à l’écran ou stockage en vue de son traitement.

### <a name="user-defined-functions"></a>Fonctions définies par l’utilisateur

Pig Latin prend également en charge les fonctions définies par l'utilisateur (UDF), ce qui vous permet d'appeler des composants externes qui implémentent la logique qui est difficile à modéliser dans Pig Latin.

Pour plus d’informations sur Pig Latin, consultez les pages [Manuel de référence Pig Latin 1](https://archive.cloudera.com/cdh/3/pig/piglatin_ref1.html) et [Manuel de référence Pig Latin 2](https://archive.cloudera.com/cdh/3/pig/piglatin_ref2.html).

## <a name="example-data"></a><a id="data"></a>Exemple de données

HDInsight propose différents exemples de jeux de données, qui sont stockés dans les répertoires `/example/data` et `/HdiSamples`. Ces répertoires sont disponibles dans le stockage par défaut de votre cluster. L’exemple Pig présenté dans ce document utilise le fichier *log4j* à partir de `/example/data/sample.log`.

Chaque journal à l'intérieur du fichier est constitué d'une ligne de champs qui contient un champ `[LOG LEVEL]` pour indiquer le type et la gravité, par exemple :

```output
2012-02-03 20:26:41 SampleClass3 [ERROR] verbose detail for id 1527353937
```

Dans l'exemple précédent, le niveau de journal est ERROR.

> [!NOTE]  
> Vous pouvez également générer un fichier log4j à l'aide de l’outil de journalisation [Apache Log4j](https://en.wikipedia.org/wiki/Log4j) , puis télécharger ce fichier vers votre objet blob. Pour des instructions, consultez la page [Téléchargement de données vers HDInsight](hdinsight-upload-data.md) . Pour plus d'informations sur l'utilisation du stockage d'objets blob Azure avec HDInsight, consultez la page [Utilisation du stockage d'objets blob Azure avec HDInsight](hdinsight-hadoop-use-blob-storage.md).

## <a name="example-job"></a><a id="job"></a>Exemple de travail

Le travail Pig Latin suivant charge le fichier `sample.log` depuis le stockage par défaut de votre cluster HDInsight. Elle effectue ensuite une série de transformations qui créent un décompte du nombre de fois où chaque niveau du journal s'est produit dans les données d'entrée. Les résultats sont écrits en STDOUT.

```output
LOGS = LOAD 'wasb:///example/data/sample.log';
LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
RESULT = order FREQUENCIES by COUNT desc;
DUMP RESULT;
```

L’image suivante montre un résumé de ce qu’effectue chaque transformation sur les données.

![Représentation graphique des transformations][image-hdi-pig-data-transformation]

## <a name="run-the-pig-latin-job"></a><a id="run"></a>Exécution de la tâche Pig Latin

HDInsight peut exécuter des tâches Pig Latin de différentes façons. Utilisez la table suivante pour choisir la méthode qui vous convient, puis cliquez sur le lien pour obtenir une présentation détaillée.

## <a name="pig-and-sql-server-integration-services"></a>Pig et SQL Server Integration Services

Vous pouvez utiliser les services SQL Server Integration Services (SSIS) pour exécuter une travail Pig. Le pack de fonctionnalités Azure pour SSIS fournit les composants suivants, compatibles avec les tâches Pig sur HDInsight.

* [Tâche Pig Azure HDInsight][pigtask]

* [Gestionnaire de connexions d’abonnement Azure][connectionmanager]

Pour en savoir plus sur le Feature Pack Azure pour SSIS, cliquez [ici][ssispack].

## <a name="next-steps"></a><a id="nextsteps"></a>Étapes suivantes

Maintenant que vous avez vu comment utiliser Pig avec HDInsight, utilisez les liens suivants pour découvrir d'autres façons d'utiliser Azure HDInsight.

* [Téléchargement de données vers HDInsight](hdinsight-upload-data.md)
* [Utilisation d’Apache Hive avec HDInsight](./hadoop/hdinsight-use-hive.md)
* [Utiliser Apache Sqoop avec HDInsight](./hadoop/hdinsight-use-sqoop.md)
* [Utilisation des tâches MapReduce avec HDInsight](./hadoop/hdinsight-use-mapreduce.md)

[apachepig-home]: https://pig.apache.org/
[putty]: https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
[curl]: https://curl.haxx.se/
[pigtask]: /sql/integration-services/control-flow/azure-hdinsight-pig-task?viewFallbackFrom=sql-server-2014
[connectionmanager]: /sql/integration-services/connection-manager/azure-subscription-connection-manager?viewFallbackFrom=sql-server-2014
[ssispack]: /sql/integration-services/azure-feature-pack-for-integration-services-ssis?viewFallbackFrom=sql-server-2014
[hdinsight-admin-powershell]: hdinsight-administer-use-powershell.md

[hdinsight-use-hive]:../hdinsight-use-hive.md

[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-submit-jobs]:submit-apache-hadoop-jobs-programmatically.md#mapreduce-sdk

[Powershell-install-configure]: /powershell/azure/

[powershell-start]: https://technet.microsoft.com/library/hh847889.aspx


[image-hdi-pig-data-transformation]: ./media/use-pig/hdi-data-transformation.gif