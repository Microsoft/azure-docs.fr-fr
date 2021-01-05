---
title: Gestion de l’espace disque dans Azure HDInsight
description: Étapes de détection de problèmes et résolutions possibles pour la gestions des problèmes d’espace disque rencontrés lors de l’interaction avec des clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 02/17/2020
ms.openlocfilehash: 82044796cca3e135c15e750d6706fe504622ebdb
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92540344"
---
# <a name="manage-disk-space-in-azure-hdinsight"></a>Gestion de l’espace disque dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="hive-log-configurations"></a>Configurations des journaux Hive

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`, où `CLUSTERNAME` est le nom de votre cluster.

1. Accédez à **Hive** > **Configurations** > **Avancé** > **Advanced hive-log4j** . Passez en revue les paramètres suivants :

    * `hive.root.logger=DEBUG,RFA`. Il s’agit de la valeur par défaut ; modifiez le [niveau de journalisation](https://logging.apache.org/log4j/2.x/log4j-api/apidocs/org/apache/logging/log4j/Level.html) sur `INFO` pour imprimer moins d’entrées de journaux.

    * `log4jhive.log.maxfilesize=1024MB`. Il s’agit de la valeur par défaut ; modifiez-la si vous le souhaitez.

    * `log4jhive.log.maxbackupindex=10`. Il s’agit de la valeur par défaut ; modifiez-la si vous le souhaitez. Si le paramètre a été omis, les fichiers journaux générés sont infinis.

## <a name="yarn-log-configurations"></a>Configurations des journaux YARN

Passez en revue les configurations suivantes :

* Apache Ambari

    1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`, où `CLUSTERNAME` est le nom de votre cluster.

    1. Accédez à **Hive** > **Configurations** > **Avancé** > **Resource Manager** . Vérifiez que la case **Activer l’agrégation de journaux** est cochée. Si elle est décochée, les nœuds de nom conservent les journaux localement et ne les agrègent pas dans le magasin distant à la fin ou à l’arrêt de l’application.

* Assurez-vous que la taille du cluster est appropriée pour la charge de travail. La charge de travail a peut-être été modifiée récemment ou le cluster a peut-être été redimensionné. [Montez en puissance](../hdinsight-scaling-best-practices.md) le cluster pour qu’il corresponde à une charge de travail plus élevée.

* `/mnt/resource` peut également être rempli de fichiers orphelins (comme dans le cas du redémarrage de Resource Manager). Si nécessaire, nettoyez manuellement `/mnt/resource/hadoop/yarn/log` et `/mnt/resource/hadoop/yarn/local`.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).