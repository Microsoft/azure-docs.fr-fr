---
title: L’espace disque est insuffisant sur le nœud de cluster dans Azure HDInsight
description: Dépannage des problèmes d'espace disque des nœuds d’un cluster Apache Hadoop dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 04/30/2020
ms.openlocfilehash: f1a994ad07980c67e37d00bffb7e605ed610bb08
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289077"
---
# <a name="scenario-cluster-node-runs-out-of-disk-space-in-azure-hdinsight"></a>Scénario : L’espace disque est insuffisant sur le nœud de cluster dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Un travail peut échouer avec un message d’erreur semblable à celui-ci : `/usr/hdp/2.6.3.2-14/hadoop/libexec/hadoop-config.sh: fork: No space left on device.`

Vous pouvez également recevoir une alerte Apache Ambari similaire à ce qui suit : `local-dirs usable space is below configured utilization percentage`.

## <a name="cause"></a>Cause

Le cache d’application Apache Yarn peut avoir consommé tout l’espace disque disponible. Votre application Spark s’exécute probablement de manière inefficace.

## <a name="resolution"></a>Résolution

1. Utilisez l’interface utilisateur Ambari pour déterminer quel nœud manque d’espace disque.

1. Déterminez quel dossier du nœud en difficulté contribue à la plus grande partie de l'espace disque. Commencez par exécuter SSH sur le nœud, puis exécutez `df` pour répertorier l’utilisation du disque pour tous les montages. En général, `/mnt` est un disque temporaire utilisé par OSS. Vous pouvez entrer dans un dossier, puis taper `sudo du -hs` pour afficher les tailles de fichiers résumés sous un dossier. Si vous voyez un dossier similaire à `/mnt/resource/hadoop/yarn/local/usercache/livy/appcache/application_1537280705629_0007`, cela signifie que l’application est toujours en cours d’exécution. Cela peut être dû à la persistance RDD ou aux fichiers de lecture aléatoire intermédiaires.

1. Pour atténuer le problème, arrêtez l’application, qui libérera l’espace disque utilisé par cette application.

1. Si le problème se produit fréquemment sur les nœuds Worker, vous pouvez régler les paramètres du cache local YARN sur le cluster.

    Ouvrez l’interface utilisateur Ambari, puis accédez YARN --> Configurations --> Avancé.  
    Ajoutez les deux propriétés suivantes à la section yarn-site personnalisé du .xml, puis enregistrez :

    ```
    yarn.nodemanager.localizer.cache.target-size-mb=2048
    yarn.nodemanager.localizer.cache.cleanup.interval-ms=300000
    ```

1. Si la solution ci-dessus ne résout pas définitivement le problème, optimisez votre application.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]