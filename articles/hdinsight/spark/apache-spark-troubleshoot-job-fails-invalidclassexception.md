---
title: Erreur InvalidClassException d’Apache Spark – Azure HDInsight
description: La tâche Apache Spark échoue avec InvalidClassException, incompatibilité de version de classe, dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/29/2019
ms.openlocfilehash: 6220c328d05e7cd68460b7bfd0708a9d393a290f
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287892"
---
# <a name="apache-spark-job-fails-with-invalidclassexception-class-version-mismatch-in-azure-hdinsight"></a>La tâche Apache Spark échoue avec InvalidClassException, incompatibilité de version de classe, dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Apache Spark dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Vous essayez de créer une tâche Apache Spark dans un cluster Spark 2.x. La tâche échoue avec une erreur semblable à la suivante :

```
18/09/18 09:32:26 WARN TaskSetManager: Lost task 0.0 in stage 1.0 (TID 1, wn7-dev-co.2zyfbddadfih0xdq0cdja4g.ax.internal.cloudapp.net, executor 4): java.io.InvalidClassException:
org.apache.commons.lang3.time.FastDateFormat; local class incompatible: stream classdesc serialVersionUID = 2, local class serialVersionUID = 1
        at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:699)
        at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1885)
        at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1751)
        at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2042)
        at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)
```

## <a name="cause"></a>Cause :

Cette erreur peut être due à l’ajout d’un fichier jar supplémentaire à la configuration `spark.yarn.jars`, spécifiquement un fichier jar grisé qui inclut une version différente du package `commons-lang3` et qui introduit une incompatibilité de classe. Par défaut, Spark 2.1/2/3 utilise la version 3.5 de `commons-lang3`.

> [!TIP]
> Ombrer une bibliothèque consiste à placer son contenu dans votre propre fichier jar, en modifiant son package. Cela diffère de l’empaquetage de la bibliothèque, qui consiste à placer la bibliothèque dans votre propre fichier jar sans réempaquetage.

## <a name="resolution"></a>Résolution

Vous pouvez soit supprimer le fichier jar, soit recompiler le fichier jar personnalisé (AzureLogAppender) et utiliser [maven-shade-plugin](https://maven.apache.org/plugins/maven-shade-plugin/examples/class-relocation.html) pour déplacer les classes.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]