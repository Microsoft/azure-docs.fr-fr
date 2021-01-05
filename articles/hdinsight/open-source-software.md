---
title: Prise en charge des logiciels open source dans Azure HDInsight
description: Microsoft Azure fournit un niveau général de prise en charge pour les technologies open source.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: fe9ccc50d6196997863fbf228fbb76913a0d0330
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013868"
---
# <a name="open-source-software-support-in-azure-hdinsight"></a>Prise en charge des logiciels open source dans Azure HDInsight

Le service Microsoft Azure HDInsight utilise un environnement de technologies open source formées autour d’Apache Hadoop. Microsoft Azure fournit un niveau général de prise en charge pour les technologies open source. Pour plus d’informations, consultez la section **Étendue du support** du [Forum aux questions sur le support technique Azure](https://azure.microsoft.com/support/faq/). Le service HDInsight fournit un niveau supplémentaire de prise en charge pour les composants intégrés.

## <a name="components"></a>Components

Deux types de composants open source sont disponibles dans le service HDInsight :

### <a name="built-in-components"></a>Composants intégrés

Ces composants sont préinstallés sur les clusters HDInsight et fournissent les fonctionnalités principales du cluster. Les composants suivants appartiennent à cette catégorie :

* [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) Resource Manager.
* Le langage de requête Hive [HiveQL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual).
* [Apache Mahout](https://mahout.apache.org/).

La liste complète des composants de cluster est disponible dans [Quels sont les composants et versions Apache Hadoop disponibles avec HDInsight ?](hdinsight-component-versioning.md)

### <a name="custom-components"></a>Composants personnalisés

En tant qu’utilisateur du cluster, vous pouvez installer ou utiliser dans votre charge de travail tout composant qui est disponible dans la communauté ou que vous avez créé.

> [!WARNING]  
> Les composants fournis avec le cluster HDInsight sont entièrement pris en charge. Le support Microsoft vous aide à isoler et à résoudre les problèmes liés à ces composants.
>
> Les composants personnalisés bénéficient d’un support commercialement raisonnable pour vous aider à résoudre le problème. Le support Microsoft peut réussir à résoudre le problème. Ou alors, il peut vous inviter à faire appel à d’autres canaux qui offrent une expertise reconnue sur ces technologies open source. De nombreux sites de communauté sont disponibles, comme par exemple la [page de questions Microsoft Q&A sur HDInsight](/answers/topics/azure-hdinsight.html) et [Stack Overflow](https://stackoverflow.com).
>
> Par ailleurs, les projets Apache ont des sites de projet sur le [site web Apache](https://apache.org). [Hadoop](https://hadoop.apache.org/) en est un exemple.

## <a name="component-usage"></a>Usage des composants

Le service HDInsight fournit plusieurs méthodes d’utilisation de ces composants personnalisés. Quel que soit le mode d’utilisation ou d’installation du composant sur le cluster, le même niveau de support s’applique. Le tableau suivant décrit les méthodes les plus courantes d’utilisation des composants personnalisés sur des clusters HDInsight :

|Usage |Description |
|---|---|
|Envoi de tâche|Il est possible d’envoyer au cluster des travaux Hadoop ou d’autres types de travaux qui exécutent ou utilisent des composants personnalisés.|
|Personnalisation des clusters|Quand vous créez le cluster, vous pouvez définir des paramètres supplémentaires et des composants personnalisés qui sont installés sur les nœuds du cluster.|
|Exemples|Pour les composants personnalisés fréquemment utilisés, Microsoft et d’autres éditeurs fournissent parfois des exemples illustrant leur utilisation sur des clusters HDInsight. Ces exemples sont fournis sans support.|

## <a name="next-steps"></a>Étapes suivantes

* [Personnaliser des clusters Azure HDInsight à l’aide d’actions de script](./hdinsight-hadoop-customize-cluster-linux.md)
* [Développer des scripts d’action de script pour HDInsight](hdinsight-hadoop-script-actions-linux.md)
* [Gérer en toute sécurité l’environnement Python sur Azure HDInsight avec une action de script](./spark/apache-spark-python-package-installation.md)