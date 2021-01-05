---
title: Apache Spark lent lorsque le stockage Azure HDInsight contient de nombreux fichiers
description: Un travail Apache Spark s’exécute lentement quand le conteneur de stockage Azure contient un grand nombre de fichiers dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/21/2019
ms.openlocfilehash: f14e9bfad959d7f40b2d364c702983c31e0e3cef
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289765"
---
# <a name="apache-spark-job-run-slowly-when-the-azure-storage-container-contains-many-files-in-azure-hdinsight"></a>Un travail Apache Spark s’exécute lentement quand le conteneur de stockage Azure contient de nombreux fichiers dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Apache Spark dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Pendant l’exécution d’un cluster HDInsight, le travail Apache Spark qui écrit dans le conteneur de stockage Azure ralentit en présence de nombreux fichiers/sous-dossiers. Par exemple, le travail dure 20 secondes lors de l’écriture dans un nouveau conteneur, mais environ 2 minutes lors de l’écriture dans un conteneur comprenant 200 000 fichiers.

## <a name="cause"></a>Cause :

Il s’agit d’un problème connu avec Spark. La lenteur est due aux opérations `ListBlob` et `GetBlobProperties` pendant l’exécution du travail Spark.

Pour suivre les partitions, Spark doit gérer un `FileStatusCache` qui contient des informations sur la structure de répertoires. À l’aide de ce cache, Spark peut analyser les chemins et connaître les partitions disponibles. Avec le suivi des partitions, Spark accède uniquement aux fichiers nécessaires quand vous lisez des données. Pour garder ces informations à jour, quand vous écrivez de nouvelles données, Spark doit lister tous les fichiers sous le répertoire et mettre à jour ce cache.

Dans Spark 2,1, même si nous n’avons pas besoin de mettre à jour le cache après chaque écriture, Spark vérifie si une colonne de partition existante correspond à celle proposée dans la demande d’écriture actuelle. Cette opération entraîne de nombreuses opérations de listing au début de chaque écriture.

Dans Spark 2,2, quand des données sont écrites avec le mode Append, ce problème de performances doit être résolu.

## <a name="resolution"></a>Résolution

Quand vous créez un jeu de données partitionné, il est important d’utiliser un schéma de partitionnement qui limite le nombre de fichiers que Spark doit lister pour mettre à jour le `FileStatusCache`.

Pour chaque énième micro-lot où N % 100 == 0 (100 n’est qu’un exemple), déplacez les données existantes vers un autre répertoire, qui peut être chargé par Spark.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]