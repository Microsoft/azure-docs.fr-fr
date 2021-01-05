---
title: Le réducteur est lent dans Azure HDInsight
description: Le réducteur est lent dans Azure HDInsight à partir d’une éventuelle distorsion des données
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/30/2019
ms.openlocfilehash: 74e5214b304870b48e6c6f3ec34b7a5bae0e389a
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93288855"
---
# <a name="scenario-reducer-is-slow-in-azure-hdinsight"></a>Scénario : Le réducteur est lent dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Interactive Query dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Lorsque vous exécutez une requête comme `insert into table1 partition(a,b) select a,b,c from table2`, le plan de requête démarre un ensemble de regroupements, mais les données de chaque partition sont dirigées vers un seul réducteur. La requête est alors aussi lente que le temps pris par le réducteur de la plus grande partition.

## <a name="cause"></a>Cause

Ouvrez [beeline](../hadoop/apache-hadoop-use-hive-beeline.md) et vérifiez la valeur du `hive.optimize.sort.dynamic.partition`.

La valeur de cette variable est censée être définie sur true/false en fonction de la nature des données.

Si les partitions de la table d’entrée sont moins nombreuses (moins de 10), de même que le nombre de partitions de sortie, et que la variable est définie sur `true`, cela entraîne le tri global et l’écriture des données à l’aide d’un seul réducteur par partition. Même si le nombre de réducteurs disponibles est plus important, il est possible que quelques réducteurs soient en retard en raison de la distorsion des données et que le parallélisme maximal ne puisse pas être atteint. Une fois passé à `false`, plusieurs réducteurs peuvent traiter une seule partition et plusieurs fichiers plus petits sont écrits, ce qui accélère l’insertion. Cela peut affecter les requêtes ultérieures en raison de la présence de fichiers plus petits.

La valeur `true` est logique lorsque le nombre de partitions est plus grand et que les données ne sont pas inclinées. Dans ce cas, le résultat de la phase de mappage sera écrit de sorte que chaque partition sera gérée par un réducteur unique, ce qui entraînera de meilleures performances pour les requêtes ultérieures.

## <a name="resolution"></a>Résolution

1. Essayez de repartitionner les données pour normaliser en plusieurs partitions.

1. Si #1 n’est pas possible, définissez la valeur de la configuration sur false dans la session Beeline et recommencez la requête. `set hive.optimize.sort.dynamic.partition=false`. La définition de la valeur sur false au niveau du cluster n’est pas recommandée. La valeur de `true` est optimale et définit le paramètre selon les besoins en fonction de la nature des données et de la requête.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]