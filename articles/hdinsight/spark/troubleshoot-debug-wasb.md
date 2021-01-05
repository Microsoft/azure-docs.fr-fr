---
title: Déboguer les opérations de fichiers WASB dans Azure HDInsight
description: Décrit les étapes de détection de problèmes et résolutions possibles pour les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 02/18/2020
ms.openlocfilehash: 85e974d51f49bbb6742683ed253c077bb3ff69de
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014650"
---
# <a name="debug-wasb-file-operations-in-azure-hdinsight"></a>Déboguer les opérations de fichiers WASB dans Azure HDInsight

Vous souhaiterez parfois comprendre quelles opérations le pilote WASB a démarré avec Stockage Azure. Pour le côté client, le pilote WASB génère des journaux pour chaque opération de système de fichiers au niveau **DEBUG**. Le pilote WASB utilise log4j pour contrôler le niveau de journalisation, et la valeur par défaut est le niveau **INFO**. Pour les journaux d’analytique côté serveur de Stockage Azure, consultez [Journalisation Azure Storage Analytics](../../storage/common/storage-analytics-logging.md).

Un journal généré ressemble à ce qui suit :

```log
18/05/13 04:15:55 DEBUG NativeAzureFileSystem: Moving wasb://xxx@yyy.blob.core.windows.net/user/livy/ulysses.txt/_temporary/0/_temporary/attempt_20180513041552_0000_m_000000_0/part-00000 to wasb://xxx@yyy.blob.core.windows.net/user/livy/ulysses.txt/part-00000
```

## <a name="turn-on-wasb-debug-log-for-file-operations"></a>Activer le journal de débogage WASB pour les opérations de fichiers

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/SPARK2/configs`, où `CLUSTERNAME` est le nom de votre cluster Spark.

1. Accédez à **advanced spark2-log4j-properties**.

    1. Remplacez `log4j.appender.console.Threshold=INFO` par `log4j.appender.console.Threshold=DEBUG`.

    1. Ajoutez `log4j.logger.org.apache.hadoop.fs.azure.NativeAzureFileSystem=DEBUG`.

1. Accédez à **Advanced livy2-log4j-properties**.

    Ajoutez `log4j.logger.org.apache.hadoop.fs.azure.NativeAzureFileSystem=DEBUG`.

1. Enregistrez les changements.

## <a name="additional-logging"></a>Journalisation supplémentaire

Les journaux ci-dessus offrent une vue globale des opérations du système de fichiers. S’ils ne fournissent toujours pas d’informations assez utiles, ou si vous souhaitez examiner les appels d’API de stockage Blob, ajoutez `fs.azure.storage.client.logging=true` au `core-site`. Ce paramètre active les journaux du SDK Java pour le pilote de stockage wasb, et affiche chaque appel au serveur de stockage d’objets blob. Supprimez ce paramètre après vos investigations, car il peut rapidement saturer le disque et ralentir le processus.

Si le back-end est basé sur Azure Data Lake, utilisez le paramètre log4j suivant pour le composant (par exemple, spark/tez/hdfs) :

```
log4j.logger.com.microsoft.azure.datalake.store=ALL,adlsFile
log4j.additivity.com.microsoft.azure.datalake.store=true
log4j.appender.adlsFile=org.apache.log4j.FileAppender
log4j.appender.adlsFile.File=/var/log/adl/adl.log
log4j.appender.adlsFile.layout=org.apache.log4j.PatternLayout
log4j.appender.adlsFile.layout.ConversionPattern=%p\t%d{ISO8601}\t%r\t%c\t[%t]\t%m%n
```

Recherchez les journaux dans `/var/log/adl/adl.log`.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support**. Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).